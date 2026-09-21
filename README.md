# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!
&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)


Updated Readme content :-
The operational data is stored in:
data/service_data.json

Observations from Logs and Metrics
The normal records show:

Response time CPU utilization Memory utilization INFO log level

2 records are different

Anomaly Detection Findings
Two anomalies were detected.

Anomaly 1 — 10:05

Why? :

High response time
Error log detected
Anomaly 2 — 10:06

Why? :

High response time
High CPU utilization
High memory utilization
Error log detected
Final Workflow Result
The final execution produced:

================================================== AIOps Pipeline Result
Records processed: 10 Anomalies detected: 2 Events consumed: 2

Both detected anomalies were successfully published and consumed.

Automated tests were also executed successfully:

8 passed

Issues Identified and Corrected
Issue 1 — Incorrect Event Topic

The producer and consumer were reading different topics.

I corrected it by connecting both the producer and consumer to the same topic.

Issue 2 — Incorrect Log-Level Detection

The anomaly detector initially treated WARNING logs as error logs.

I corrected it by changing "WARNING" to "ERROR"
