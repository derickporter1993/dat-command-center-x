


Command Center X Readme
Command Center X
High-visibility Salesforce operations console that surfaces governor limit consumption, Flow execution health, CI/CD deployment status, and real-time performance alerts in one admin-only workspace.

Status: Reference implementation. Deploy to scratch, sandbox, or production orgs. Last Updated: 16 Jul 2025 (PT).

Table of Contents
1. Why Command Center X Exists

2. Core Features

3. Architecture Overview

4. Component Map

5. Quick Start (Scratch Org)

6. Install to Sandbox or Production

7. Post-Install Configuration Checklist

8. Dashboards & Components Usage

8.1 Apex Governor Limits Dashboard

8.2 Flow Execution Monitor

8.3 Deployment Monitor Dashboard

8.4 Performance Alert Panel

9. Problem → Feature → Business Value Table

10. DAT-Specific Callouts (Sales Tech Use Case)

11. Configuration: Thresholds, Auto-Refresh, Alert Routing

12. Security & Access Control

13. CI/CD Integration (DevOps Center + GitHub Actions)

14. Data Model & Metadata Assets

15. Extensibility Patterns

16. Troubleshooting

17. Video Learning Index

18. FAQ

19. Contributing

20. License

1. Why Command Center X Exists
Salesforce orgs accumulate technical debt fast: unmanaged Flows, ad-hoc Apex, growing data volume, and unclear deployment health lead to slow delivery and incidents that surface too late. Command Center X centralizes real-time operational signals so administrators see issues before users do. It reduces blind spots, accelerates troubleshooting, and provides quantifiable performance metrics that leadership understands.

2. Core Features
Apex governor limit telemetry with visual CPU-over-threshold alerting (default CPU threshold 9,500 ms vs 10,000 limit).

Flow execution logging to custom object for run counts, fault rate, and top offenders.

CI/CD deployment result feeds from DevOps Center or custom Deployment records.

Platform Event based real-time alerting for high CPU, SOQL, DML breaches, or Flow faults.

Auto-refresh (default 60s) plus manual refresh control.

Admin-only visibility via dedicated permission set.

Chart.js powered visualizations (doughnut, gauge, bar).

3. Architecture Overview
External Data Layer (Mockaroo, CSV Imports)
        ↓
Salesforce Core (Accounts, Contacts, Opportunities, Freight Lane, Carrier)
        ↓
Automation Layer (Flows, Auto-Convert, Discount Approval Apex, Platform Event Triggers)
        ↓
Analytics Layer (CRM Analytics Dashboards, Tableau CRM Recipes)
        ↓
Logging Layer (Flow_Execution__c, Performance_Alert__e)
        ↓
DevOps Layer (DevOps Center Pipelines, GitHub Actions)
Color Legend used in diagrams:

Blue = Data

Green = Automation

Yellow = Analytics

Red = Logging

Gray = DevOps

4. Component Map
Directory layout as deployed via SFDX.

command-center-x/
├── README.md
├── sfdx-project.json
├── config/project-scratch-def.json
├── scripts/
│   └── orgInit.sh
└── force-app/main/default/
    ├── lwc/
    │   ├── systemMonitorDashboard/
    │   ├── flowExecutionMonitor/
    │   ├── deploymentMonitorDashboard/
    │   └── performanceAlertPanel/
    ├── classes/
    │   ├── LimitMetrics.cls
    │   ├── FlowExecutionLogger.cls
    │   └── PerformanceAlertPublisher.cls
    ├── objects/Flow_Execution__c/
    ├── events/Performance_Alert__e/
    ├── permissionsets/Command_Center_Admin.permissionset-meta.xml
    └── dashboards/ (CRM Analytics JSON configs)
5. Quick Start (Scratch Org)
Assumes Node + npm installed.

# 1. Clone
git clone https://github.com/YOUR-ORG/command-center-x.git
cd command-center-x

# 2. Auth to DevHub (once)
sfdx auth:web:login -d -a DevHub

# 3. Create scratch org
sfdx force:org:create -f config/project-scratch-def.json -a CCX -s -d 7

# 4. Push source
sfdx force:source:push -u CCX

# 5. Assign permission set
sfdx force:user:permset:assign -n Command_Center_Admin -u CCX

# 6. Import sample data (optional)
sfdx force:data:tree:import -f data/sample-plan.json -u CCX

# 7. Open org
sfdx force:org:open -u CCX -p /lightning/n/Command_Center_X
6. Install to Sandbox or Production
Minimal commands for a direct metadata deploy (no scratch).

# 1. Authorize target org
echo "Browser opens; login to target org"
sfdx auth:web:login -a TargetOrg

# 2. Deploy metadata
sfdx project deploy start -o TargetOrg
# or classic mdapi
# sfdx force:mdapi:deploy -d mdapi_out -u TargetOrg -w -1

# 3. Assign permission set
sfdx force:user:permset:assign -n Command_Center_Admin -o TargetOrg

# 4. Upload Chart.js static resource (Setup > Static Resources > New > ChartJS411.zip)

# 5. Add Command Center tab to Admin App.
7. Post-Install Configuration Checklist
Step	Required	Description	Location
Upload Chart.js static resource	Yes	Required for charts to render	Setup > Static Resources
Verify Platform Event "Performance_Alert__e"	Yes	Ensure event exists & fields deployed	Setup > Platform Events
Confirm Flow_Execution__c object	Yes	Add to Nav + grant field perms	Object Manager
Add LWC to Lightning App Page	Yes	Build a Command Center Lightning App page	Lightning App Builder
Assign Command_Center_Admin perm set	Yes	Controls visibility	Setup > Permission Sets
Configure alert subscriptions (email/Slack)	Optional	Use Flow or Apex Trigger on event	Automation Studio
Adjust thresholds (if using Custom Metadata)	Optional	CCX_Threshold__mdt	Custom Metadata Types
8. Dashboards & Components Usage
Each LWC reads from an Apex controller and refreshes on interval (default 60s). All LWCs include manual Refresh buttons.

8.1 Apex Governor Limits Dashboard
Component: systemMonitorDashboard Source Apex: LimitMetrics.fetchGovernorStats() Displays CPU time, heap, SOQL count, DML count. Highlights red if CPU > 9,500 ms.

8.2 Flow Execution Monitor
Component: flowExecutionMonitor Source Apex: query Flow_Execution__c aggregated by Flow name, fault count, last run. Instrument Flows by inserting an Apex Action call to FlowExecutionLogger.log.

8.3 Deployment Monitor Dashboard
Component: deploymentMonitorDashboard Source Apex: query DevOps Center Deployment records or custom object Deployment__c (map to stage, duration, success/fail count). Surfaces promotion velocity.

8.4 Performance Alert Panel
Component: performanceAlertPanel Listens to Platform Event Performance_Alert__e. Shows live scrolling feed of events above configured thresholds (CPU, SOQL, DML, Flow faults). Optional subscribe-to-Slack automation.

9. Problem → Feature → Business Value Table
Problem	CCX Feature	Value (Time / Risk / Adoption)
CPU spikes crash user transactions without warning	Red threshold alert + real-time Platform Event	Faster incident response; protects revenue workflows
Unknown Flow fault rate	Flow Execution logging + fault %	Targeted remediation; reduces user-report volume
Slow release cadence; failed deployments	Deployment Monitor pipeline telemetry	Shorter lead time for change; fewer rollback hours
Admins lack unified signal	Single Lightning App page with 4 dashboards	Shared situational awareness; better cross-team coordination
10. DAT-Specific Callouts (Sales Tech Use Case)
DAT cares about: sales velocity, data quality across carriers/lanes, and reliable release throughput. Map CCX capabilities to these outcomes.

DAT Pain	CCX Alignment	Talking Point
Sales org slows when automation errors block lead routing	Flow Execution Monitor pinpoints bottleneck Flows; alert before routing fails	Keeps opportunities flowing to reps; protects top-line bookings
Freight & carrier data quality drifts after acquisitions	Add validation + logging Flows; track fault rate by carrier onboarding Flow	Data trust accelerates analytics & pricing accuracy
Release risk across multiple teams	Deployment Monitor shows pipeline health & failure trend	Predictable releases; compresses downtime windows
11. Configuration: Thresholds, Auto-Refresh, Alert Routing
11.1 Thresholds
Default CPU critical: 9,500 ms. Modify in JS or externalize:

Custom Metadata Type: CCX_Threshold__mdt fields CPU__c, Heap__c, SOQL__c, DML__c.

Retrieve via Apex utility CCX_Config.getThreshold('CPU').

11.2 Auto-Refresh Interval
Default 60s. Change setInterval() value in each component or expose in Custom Metadata.

11.3 Alert Routing
Subscribe to Platform Event Performance_Alert__e using:

Flow (Record-Triggered on Event Bus)

Apex Trigger to send email, Slack webhook, Teams, PagerDuty

Mulesoft or external listener via CometD

12. Security & Access Control
Permission Set: Command_Center_Admin grants object, field, and event access plus LWC visibility.

Lightning App visibility filtered to users with perm set.

Shield Event Monitoring customers can extend CCX by ingesting EventLogFile metrics.

Restrict Platform Event subscription tokens in external middleware.

13. CI/CD Integration (DevOps Center + GitHub Actions)
13.1 DevOps Center Flow
Project in DevOps Center mapped to GitHub repo.

Work items track metadata changes; promote through Environments (Dev, UAT, Prod).

Deployment Monitor queries DevOps Center records (or custom mirror object) to render pass/fail by stage and average cycle time.

13.2 GitHub Actions Sample
name: Deploy-to-Sandbox
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: sfdx-actions/setup-sfdx@v2
      - name: Auth Sandbox JWT
        run: sfdx auth:jwt:grant --clientid ${{ secrets.CONSUMER_KEY }} --jwtkeyfile server.key --username ${{ secrets.SF_USERNAME }} --instanceurl https://test.salesforce.com -a SANDBOX
      - name: Deploy Metadata
        run: sfdx project deploy start -o SANDBOX
14. Data Model & Metadata Assets
14.1 Custom Object: Flow_Execution__c
Fields: Flow_Name__c (Text), Primary_Record__c (Lookup), Run_Time__c (DateTime), Status__c (Picklist Success|Fault), CPU__c (Number), SOQL__c (Number), DML__c (Number).

14.2 Platform Event: Performance_Alert__e
Fields: Metric__c, Value__c, Threshold__c, Severity__c (Formula), Context_Record__c (Text 18), Stack__c (Long Text 131k optional).

14.3 Optional Custom Object: Deployment__c (if not using DevOps Center API)
Fields: Stage__c, Result__c, Duration_Seconds__c, Components__c, Timestamp__c.

15. Extensibility Patterns
Add Health Check ingestion: nightly batch reads Setup Health Check scores into CCX.

Einstein GPT summarizer: weekly digest of top 5 CPU offenders.

Data Cloud integration: correlate high-limit usage with segments or data spikes.

Shield Event Monitoring: feed API transaction counts into charts.

Installable AppExchange packaging: convert to unlocked package for upgrade paths.

16. Troubleshooting
Symptom	Likely Cause	Fix
Blank charts	Missing Chart.js static resource	Upload zip; clear cache; reload
Data never updates	Apex errors or missing permission set	Check Debug Logs; assign Command_Center_Admin
Alert Panel empty	No Platform Events published	Confirm PerformanceAlertPublisher.publish() is called
Flow Monitor empty	Flows not instrumented	Add Apex Action to Flows calling FlowExecutionLogger.log
Deployment Monitor stale	No DevOps Center connection	Configure API user or custom Deployment__c feed
17. Video Learning Index
Verified July 2025.

Topic	Video	Notes
Create Developer Edition Org	https://www.youtube.com/watch?v=7HJAbMSaoeE	Fast free org setup
Install Salesforce CLI	https://www.youtube.com/watch?v=RUOvKct2sxI	CLI install guide
Git Basics	https://www.youtube.com/watch?v=HVsySz-h9r4	Repo fundamentals
Push to Scratch Org	https://www.youtube.com/watch?v=3Gd47-i5_dw	Source push walkthrough
Assign Permission Set	https://www.youtube.com/watch?v=ev7Rf3COVxc	UI + CLI steps
Alt Dev Org Video	https://www.youtube.com/watch?v=BYdqjtw6GO4	Backup tutorial
Alt Dev Org Deep Dive	https://www.youtube.com/watch?v=T7aWFbN-EJg	Extended version
SaaSGuru Dev Org	https://www.youtube.com/watch?v=02p5yfqbntQ	Detailed setup
18. FAQ
Q: Do I need Shield? A: No. Shield adds extended telemetry but is optional.

Q: Can non-admins see the dashboards? A: Only if granted the permission set or if page-level security is relaxed.

Q: How do I change the CPU threshold without editing code? A: Enable the Custom Metadata Type extension (Section 11) and update values in the UI.

Q: Does this capture historical governor usage across transactions? A: Native Limits.get*() reports current-context usage. Persist historical data by writing to a custom object in triggers or scheduled jobs.

19. Contributing
Fork repo.

Create feature branch.

Add tests (LWC Jest + Apex test classes min 75% cov; cover alert publish logic > 1 event path).

Submit pull request with description + before/after screenshot.

Coding Standards:

Apex PMD clean (no critical).

LWC ESLint clean.

Naming: CCX_ for shared utilities.

20. License
MIT unless superseded by enterprise distribution agreement. See LICENSE file.

Appendix A. Representative Code Snippets
LWC: systemMonitorDashboard.js
import { LightningElement } from 'lwc';
import fetchGovernorStats from '@salesforce/apex/LimitMetrics.fetchGovernorStats';
import { loadScript } from 'lightning/platformResourceLoader';
import CHARTJS from '@salesforce/resourceUrl/ChartJS411';

export default class SystemMonitorDashboard extends LightningElement {
  chart; timer;
  renderedCallback() {
    if (this.chart) { return; }
    Promise.all([loadScript(this, CHARTJS + '/Chart.min.js')])
      .then(() => this.initChart())
      .catch(err => console.error('Chart load error', err));
  }
  initChart() {
    const ctx = this.template.querySelector('canvas').getContext('2d');
    this.chart = new window.Chart(ctx, {
      type: 'doughnut',
      data: { labels: ['CPU','Heap','SOQL','DML'], datasets: [{ data: [0,0,0,0] }] },
      options: { cutout: '60%' }
    });
    this.refresh();
    this.timer = setInterval(() => this.refresh(), 60000);
  }
  disconnectedCallback() { clearInterval(this.timer); }
  refresh() {
    fetchGovernorStats().then(stats => {
      const { cpuTime, heapSize, soqlQueries, dmlStatements } = stats;
      this.chart.data.datasets[0].data = [cpuTime, heapSize, soqlQueries, dmlStatements];
      this.chart.data.datasets[0].backgroundColor = cpuTime > 9500 ? ['#ff0000','#e0e0e0','#e0e0e0','#e0e0e0'] : undefined;
      this.chart.update();
    }).catch(err => console.error('Stats error', err));
  }
}
Apex: LimitMetrics.cls
public with sharing class LimitMetrics {
  @AuraEnabled(cacheable=true)
  public static Map<String,Integer> fetchGovernorStats() {
    return new Map<String,Integer>{
      'cpuTime' => Limits.getCpuTime(),
      'heapSize' => Limits.getHeapSize(),
      'soqlQueries' => Limits.getQueries(),
      'dmlStatements' => Limits.getDmlStatements()
    };
  }
}
Apex: FlowExecutionLogger.cls (Invocable for Flows)
public with sharing class FlowExecutionLogger {
  @InvocableMethod(label='Log Flow Execution')
  public static void log(List<Id> recordIds) {
    Flow_Execution__c fe = new Flow_Execution__c();
    fe.Flow_Name__c = Flow.Interview.getCurrentFlowName();
    fe.Primary_Record__c = recordIds.isEmpty() ? null : recordIds[0];
    fe.Run_Time__c = System.now();
    fe.Status__c = 'Success'; // override in extended impl
    insert fe;
  }
}
Apex: PerformanceAlertPublisher.cls
public with sharing class PerformanceAlertPublisher {
  public static void publish(String metric, Integer value, Integer threshold) {
    if (value > threshold) {
      Performance_Alert__e evt = new Performance_Alert__e(
        Metric__c = metric,
        Value__c = value,
        Threshold__c = threshold
      );
      EventBus.publish(evt);
    }
  }
}
Appendix B. Sample Flow Instrumentation Steps
Open Flow Builder; edit target Flow.

Add Action element.

Type = Apex. Select "Log Flow Execution" (FlowExecutionLogger.log).

Pass {!$Record.Id} or a collection depending on entry context.

Save + Activate.

Run Flow; verify record created in Flow Executions tab.

Appendix C. Lightning App Page Layout Suggestion
Region 1 (full width): Apex Governor Limits donut.

Region 2 left: Flow Execution table.

Region 2 right: Deployment status bar chart.

Region 3 (full width): Real-time Alert feed.
