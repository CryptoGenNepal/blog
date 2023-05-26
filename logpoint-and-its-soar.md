---
title: "LogPoint and its SOAR"
date: 2023-03-01T00:00:00+05:45
# post image
image: "images/blog/logpoint-and-its-soar/Topic.png"
# author
author: "Bishesh Shrestha"
# post type (regular/featured)
type: "featured"
# meta description
description: "SOAR stands for Security Orchestration, Automation, and Response. To put it in simple terms, it is like a superhero team of IT experts who work together to protect the organization from cyber threats. In simple terms, SOAR is a set of instructions and tools that help security teams work more efficiently. So why SOAR? A simple answer to this question can be that it can automate processes and alerts when suspicious activity is detected. It can also automate other tasks such as the isolation or remediation of compromised devices."
# post draft
draft: false
---

{{< toc >}}

If you are into the cyber world then you probably have heard of SIEM. Here in this blog, I will be sharing tips and how you can use LogPoint and its components of SOAR to protect your organization from various kinds of cyber threats. These all things have been collected through my experience using the product itself. So long story short

## LogPoint Introduction

LogPoint is a SIEM (security information and event management) solution that is designed to help organizations detect and respond to security threats in real-time by collecting, analyzing and correlating log data from a variety of logs sources. There are also many functionalities that it provides, such as Log management, Advanced analytics, Visualization, reporting, alerting, integration and SOAR. To put it in simple terms, It is a comprehensive security solution which provides advanced threat detection, and automated incident response capabilities to help organizations detect and respond to the corresponding threats.

So, you might have heard of all these things about how every incident is detected and responded to automatically and the things that happen behind the scene of the organization which leads our question to one point.

## What is SOAR?

SOAR stands for Security Orchestration, Automation, and Response. To put it in simple terms, it is like a superhero team of IT experts who work together to protect the organization from cyber threats. In simple terms, SOAR is a set of instructions and tools that help security teams work more efficiently. So why SOAR? A simple answer to this question can be that it can automate processes and alerts when suspicious activity is detected. It can also automate other tasks such as the isolation or remediation of compromised devices.

SOAR is a system that helps organizations improve incident response time and effectiveness by automating tasks, coordinating actions and providing a straightforward incident management process. It makes the incident response process more efficient, by reducing manual work and improving communication and collaboration among the teams involved.

Today, I will delve into the technical aspects of the detection procedure, specifically focusing on the implementation of Auto-Case Creation. This will enable efficient and automated detection of incidents for Response. In a follow-up writeup, I will then examine the utilization of this information to facilitate automated response mechanisms of SOAR.

## Detection

Automated investigation refers to the use of software tools to streamline the process of analyzing incidents and determining an appropriate response. In this context, the use of cases in LogPoint is emphasized as a way to create automatic cases from detected alerts. The goal is to automate the investigation process by automatically creating cases based on the detection of alerts, thus allowing the security orchestration, automation, and response (SOAR) system to quickly respond to incidents. By doing this, manual efforts are reduced, enabling the security team to focus on more critical tasks.

## Installation

Imagine this, you have a Windows machine that is constantly generating logs and events. You want to put these logs to good use and leverage them for security purposes. That’s where LogPoint comes in! By installing the LP agent on your Windows machine, you’re able to forward all those logs to a centralized location, our trusty Security Information and Event Management (SIEM) system. With just a few clicks, you’ll be able to transform your logs into valuable insights that can help you respond to incidents quickly and efficiently. Think of it as a virtual detective that’s working 24/7 to keep your environment secure. Want to give it a try? Check out the installation guide linked below:

[https://docs.logpoint.com/docs/logpoint-agent/en/latest/](https://docs.logpoint.com/docs/logpoint-agent/en/latest/)

{{< image src="images/blog/logpoint-and-its-soar/1.png" caption="Status of agent" alt="Status of agent" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Status of agent" >}}

After installation, our LP agent in our windows machine is up and running.

## Use case and trigger
There can be a lot of use cases for SOAR, some of which could be like  **blocking an account from Active Directory**  if it is being attacked with brute force technique or it could also be something related to  **File Integrity Monitoring (FIM)**  in which if there is a certain file drop in the system our SOAR could instantly respond to it before any Security analyst even discovers it. This will help analysts to be more efficient and the SIEM solution to be more effective.

For this example, my use case will be event id  **4625** which is a password failure or bad password. I am keeping it simple since it will be easy to understand and can be triggered easily.

> _Note: You can use according to your own necessities_

So here I have created an alert rule which will trigger once there is an occurrence of event id 4625 which is a bad password.

{{< image src="images/blog/logpoint-and-its-soar/2.png" caption="Alert Rule for password failure" alt="Alert Rule for password failure" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Alert Rule for password failure" >}}

For every password failure coming from the windows machine, this alert will be triggered which will create an incident.

{{< image src="images/blog/logpoint-and-its-soar/3.png" caption="Log for the incident" alt="Log for the incident" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Log for the incident" >}}

{{< image src="images/blog/logpoint-and-its-soar/4.png" caption="Incident Created from Alert" alt="Incident Created from Alert" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Incident Created from Alert" >}}

{{< image src="images/blog/logpoint-and-its-soar/5.png" caption="Alert Incident Data" alt="Alert Incident Data" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Alert Incident Data" >}}

In the alert incident data, we will get the AlertRule ID. This will be handy afterwards when automating the investigation.

## Playbooks
Playbooks in LogPoint are automated scripts that execute a series of tasks and actions to address specific security incidents or scenarios. They help organizations to standardize their security operations and improve the efficiency and speed of response to security incidents. Playbooks can be triggered manually or automatically based on specific events or conditions and can include actions such as data collection, correlation, analysis, and remediation. They can be customized to meet the specific needs and requirements of an organization and can integrate with other security tools and systems to provide a comprehensive and unified approach to security operations. Here for my use case, I have created a playbook named ‘**Password Failure**’ which will create a case and run sub-playbooks:

{{< image src="images/blog/logpoint-and-its-soar/6.png" caption="Playbook" alt="Playbook" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Playbook" >}}

The next step here will be triggering this playbook automatically from a generated alert. So for triggering the playbooks automatically LogPoint SOAR provides something called  **Playbook Trigger (Automation).**

## Playbook Trigger (Automation)
For creating an automated playbook trigger we first need to go to the trigger tab and click on  **+Create Trigger.**

{{< image src="images/blog/logpoint-and-its-soar/7.png" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Then in the General tab name the playbook trigger and put the source as  **LogPoint.** After that give a description according to your use case and set the severity label and click on enable.

{{< image src="images/blog/logpoint-and-its-soar/8.png" caption="General Tab in Playbook Trigger" alt="General Tab in Playbook Trigger" height="" width="" position="center" command="fit" option="" class="img-fluid" title="General Tab in Playbook Trigger" >}}

> Remember we got the alertrule_id from the incident data which was created before.

Now go into the Trigger tab and write a trigger query something like this:

{{< image src="images/blog/logpoint-and-its-soar/9.png" caption="Trigger Query in Playbook Trigger" alt="Trigger Query in Playbook Trigger" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Trigger Query in Playbook Trigger" >}}

What this will do is trigger the playbook once there is the alertrule_id match in the LogPoint Table.

Lastly, for the Automation part go to the Automation tab and select the playbook you need to automate once the alert is triggered. Since we have created a playbook which will automatically create a case item. This whole procedure should do the same.

{{< image src="images/blog/logpoint-and-its-soar/10.png" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

Now we can see in the monitoring tab of the playbooks that the playbook named ‘**Password Failure**’ is executed successfully when the incident is triggered:

{{< image src="images/blog/logpoint-and-its-soar/11.png" caption="Monitoring Tab for Playbook" alt="Monitoring Tab for Playbook" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Monitoring Tab for Playbook" >}}

All in all, Our goal for automated case creation is completed.

{{< image src="images/blog/logpoint-and-its-soar/12.png" caption="Case Creation" alt="Case Creation" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Case Creation" >}}

## Conclusion
In conclusion, we’ve just scratched the surface of the capabilities of LogPoint SOAR’s detection and investigation features. It’s a powerful tool that streamlines the process of detecting potential security threats and conducting thorough investigations. But the real magic happens when we move on to the next chapter and explore the world of automated response. With the ability to quickly and efficiently respond to incidents, LogPoint SOAR takes the security game to a whole new level. So, keep your eyes peeled for our next blog, where we’ll dive even deeper into the wonders of this cutting-edge technology.