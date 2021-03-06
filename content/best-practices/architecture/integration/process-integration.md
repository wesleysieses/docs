---
title: "Process Integration"
parent: "integration-solutions"
menu_order: 6
draft: true
---

## 1 What Is Process Integration?

Process integration is a wide and important area that promises automation, digitization, and operational improvement. Often there is a significant business case for this type of IT development, where companies can gain on their competition by being more efficient, running lower costs, and providing better and faster services.

When a business process runs across several systems, there will be some type of process integration. There are many ways this can be done:

* **Business event integration** – Work finishes in one app, and the next app is notified to start the next step of the process automatically. This avoids the need to, for example, send emails and retype information into another system.

	![](attachments/process-integration/process-int1.png)

* **Workflow integration** – A user works in one app and then continues the same process in another app (for example, via a deep link). By enabling this integration, you can have specialized apps or microservices that evolve separately, but for the end-user it seems to be the same system. For more information, refer to [Example – Workflow Integration with Data Transfer](workflow-int-data-transfer).

	![](attachments/process-integration/process-int2.png)

* **Case management** – Human workflow in phases maintains a “case” object with data and status. The case is routed to different user-group baskets and/or sub-cases are created and completed in other systems. This is the best way to support a cross-departmental process, where several user groups are involved with approvals and coordination.

	![](attachments/process-integration/process-int4.png)

* **Process orchestration** – A system actively orchestrates a process across several systems, keeping track of status, retrying when required, and escalating to human workflow when required. This is useful for automating transaction processing (for example, in banks), for provisioning a bill of materials, and when a business event should lead to updates in several systems in parallel.
	
	![](attachments/process-integration/process-int3.png)

* **State engine/event manager** – Events are passively gathered related to a process that runs across several systems. This is to determine that the process finishes correctly and that actions can be taken when something is wrong. This is useful for monitoring and managing a chain of business events (for example, packages for track & trace) and for high-volume asynchronous process orchestration.

	![](attachments/process-integration/process-int5.png)

## 2 Business Events & Process Flow {#business}

The most common process integration is for business events. This means that some part of a  process finishes in one app, which triggers something to happen in the next app or system. The business-event messages can be transferred in different ways, depending on the requirements. 

These are the most common options:

* **REST push end-to-end** – this is the best option when the destination system should validate the business event, especially when there is a user working in the source system who can correct information directly
* **REST pull from the next system** – this is the simplest and most commonly used option, which works well for a majority of situations
* **Queues in the middle** – you can use a message broker or any other external queuing system when there is a high volume, large distance, or poor network

This diagram presents the three main options, and there is further description below:

![](attachments/process-integration/process-int6.png)

* **When validation is required in destination, use a REST push to the next system**<br />
	* App 1 finishes a piece of work pushes the business event forward, which is a good idea when a human worked on the case and if app 2 needs to verify that the data is correct. By doing this synchronously, the user can correct data directly.
* **With simple and low volume, use a REST pull from the next system**<br />
	1. App 1 finishes a piece of work and a flag is set or an internal event is created and stored internally. App 2 polls app 1 to ask for new work. When it has correctly picked up a piece of work, it resets the flag.<br />
	2. If a functional exception occurs in app 2, it may need to notify app 1 and ask for a correction using a separate service.
* **For high volume, large distance, or poor network, use a message broker in the middle or any other external queue**<br />
	1. App 1 finishes a piece of work and pushes an event to the external queue.<br />
	2. App 2 polls the queue to ask for new work, committing from the queue when safely stored.<br />
	3. If a functional exception occurs in app 2, it may need to notify app 1, which then often also goes via a response queue.<br />
	4. To minimize the number of errors in the destination, a technical XSD validation can be made in app 1 before sending, minimizing errors in the destination.

### 2.1 Selecting the Best Option

A REST pull from app 2 to app 1 (the second option above) should be the default option. This is the most robust option, and both apps can be redeployed independently.

The REST push option is good when you want app 2 to validate the business event while making sure that app 1 and app 2 are always in sync. However, this means you may not be able to finish the process in app 1 when app 2 is down.

Queues should be used in some cases, for example, with distributed networks or IoT-style integration. It can also be used along with an event manager for automated straight-through processing (for details, see the , see the [Event Managers](#event-managers) section below).  
	
Kafka is often used as an advanced queueing system for one-way communications, such as IoT, central logging, or any other situation where there is a high volume, many-to-many distributed integration situation.

## 3 Workflow Integration

Mendix is often used as a workflow tool, where a partly manual business process is implemented in a Mendix app. The Mendix app can perform the workflow on top of SAP or legacy systems, or the app could be a departmental “business portal” that allows users to work in one single app instead of opening 10-20 different applications.

Workflow integration means that the human workflow is handled across several apps via links in the UI (which are usually deep links). The end-user is often unaware that there are two separate apps. Sometimes you need to copy data behind the scenes, so the user has new data when they come back to the original app.

For more information, see [How to Do Workflow Integration](#how-workflow-int).

### 3.1 Example 1 – Dashboard for Standard Microservices System

This pattern is common for [microservices](../microservices/microservices-overview). When the process is large and/or contains functionally separate parts, it makes sense to split the “system” into a set of microservices. There is often a dashboard app where all the end-user groups log in and where they have the right to access different parts of the process via deep links.

![](attachments/process-integration/process-int7.png)

### 3.2 Example 2 – Complex Products in Separate Apps

A real example here is for the insurance sector, as shown in the diagram below. Insurance products can be quite complicated, and by having a separate app for each product, this insurance provider can drastically shorten their time-to-market for new offerings. The end-user (insurance agent or customer) works in the portal, but when selecting a product, the actual detailed ordering and configuration of that product is local and done in a separate app. When the work there is done, the end-user is redirected back to the main app. Key parts of the data are also copied over to be visible there.

![](attachments/process-integration/process-int8.png)

### 3.3 Example 3 – Creating a Customer in a Separate App

In the diagram below,  the customer app is separated from the ordering app. That is because managing customer data in this organization is a central function, and the ordering is just for one business line. The customer app has a lot of validation logic (contained in both the UX and the microflows), which we do not want to copy to the other apps.

![](attachments/process-integration/process-int10.png)

In this example, the same end-user enters both the customer information (if new) and the ordering information. You then get a workflow across two apps where data needs to be copied directly after the transaction in the second app. This means the data is used in the ordering app directly after it is created in the customer app.

This is the flow for this ordering process example:

1. The end-user creates an order in the ordering app.
2. The customer searches for the the customer in the customer app and allows the end-user to select it in the ordering app.
3. If the customer is not found, the end-user clicks **Create User** in the ordering app UI. Via a deep link, the end-user is taken to the customer app UI.
4. The customer is created and stored in the customer app, after which the end-user is linked back to the same context/order in the ordering app. Note that the data should be saved in the ordering app to be available when the end-user comes back. Alternatively, selecting the customer can be made to be the first step in the ordering process.
5. Before control is handed over to the user in the ordering app, the customer data is retrieved and stored in the relevant fields on the order. This is then displayed in the ordering app UI.

## 4 Case Management {#case}

When possible in process integration, human work is mostly done in phases where certain pre-conditions are completed first. This leads to the need for case-management solutions.

When there is a long-running cross-departmental process with many decision points and approvals, it makes sense to treat the process as a case. A case contains a set of data that is enriched through the process as a set of tasks are completed during one or more phases. The order of most tasks are arbitrary, as humans decide to work on one or more tasks at the same time. The case management tool keeps track of the pre-conditions for some tasks, the criteria for moving from one phase to the next, and the addition of tasks for certain situations.

A simple example here is a ticket management system that manages the ticket (case) through several steps. The case can be reassigned to other groups or users within the same app, or it can be sent out as sub-cases to other apps to complete.

A more complex example is a patent application or corporate loan application. This type of case goes through several phases over 6–24 months. Within each phase, a significant number of tasks are completed. Some of these tasks are done in parallel, but some require a set of other tasks to be completed first. Some tasks are automated, some are sent out to other systems for completion, and some are done by a human in the case management system itself.

The diagram below shows a case management system built as a dashboard/case-manager with work-baskets and overall control in addition to a few sub-case apps to handle specific parts of the work:

![](attachments/process-integration/process-int12.png)

In the diagram, a case is triggered from the outside or from a core system. The "intake" phase collects and validates the required data, and the case is then planned. After that, a due diligence or research phase is done in an app specializing in that. Then, a decision-maker in the case system can decide to proceed and create a proposal, which requires document creation and legal capabilities. Next, a negotiation phase occurs that is specialized in internal and external collaboration, and new proposals can be created. Finally, a contract is signed and other systems are initiated, including the core system, similar to [process orchestration](#po).

Case management solutions have taken over from BPM for process management where there is human workflow involved. This is due to the larger flexibility available for when and how work is done. However, BPM remains relevant in the area of automated process orchestration (for more information, see the [Process Orchestration](#po) section below.
	
### 4.1 Mendix for Case Management or BPM

The Mendix Platform is a good alternative to pre-defined BPM and case-management systems because the automation and case management can be adapted to the specific process problems that needs a solutions. Many existing BPM and Case management platforms were built for a specific business use-case, and they often traslate poorly to other insustries, sectors or business problems. But when using Mendix for complex process automation, we recommend using an experienced solution architect to get started. 

## 5 Process Orchestration {#po}

When a process finishes in one app and several systems need to be updated at the same time, there is a need for process orchestration. This business process runs over several systems, and you need a component that orchestrates the work done in several other areas.

In the diagram below, the support app has been given the task of process orchestration for the operationalization of a newly sold product. There are two reasons for this:

* The support app already deals with cases and distribution of tasks
* If there is an issue the customer, is likely to call Support

![](attachments/process-integration/process-int11.png)

This pattern is similar to what BPM engines do, and it works for highly automated and well-defined processes. Mendix is often used for this purpose, via an existing app (as in the diagram) or via a separate orchestration app where the UX to handle errors is easier to realize than in an ESB or a queue-management system.

## 6 Event Managers {#event-managers}

Event managers are close relatives to process orchestration. These act as passive state engines for automated processes by receiving events from all participants in a straight-through processing chain. They then pick together and monitor the status of each initiated process. 

The diagram below shows a typical situation from the logistics or supply-chain industry:

1. A chain of apps implements the phases in a mostly automated process. 
2. A queueing system or message broker is used as a transportation mechanism.
3. An event manager collects the status of each overall process using events from the apps, raising exceptions when required.
4. A track & trace dashboard app alows for checking the status of any specific automated process or getting alarms. 

![](attachments/process-integration/process-int13.png)

The event manager makes sure that messages are not lost and that processes are not halfway finished. It can raise alarms when a phase is missing, something goes wrong, or a process takes too long to finish. The result of the alarm could be to notify the correct app in the chain, or to send a message to a dashboard for monitoring the process.

Dashboards and/or track & trace solutions are usually built separately using services to retrieve the status or alarms. This makes them independent and allows for more than one-purpose views (for example, internal dashboard, external track & trace).

The event manager solution is typically used when the following factors are true:

* The process is mostly automated with straight-through processing across a chain of apps 
* There is very high volume
* Systems are distributed or have unclear uptimes
* There is another reason for using event based integration (for details, see the [State Engines & Event Managers](event-integration#state) section of *Event-Driven Integration*)

The Mendix Platform can be used for building all of the components in the diagram above, but most often Mendix is chosen for building the following:

* Automated straight-through processing apps
* Internal dashboards and control apps

## 7 How to Do Workflow Integration {#how-workflow-int}

In most architectures, there are business processes that exceed the functionality of a single microservice app. In that case, there will be some transactional data used in that process that needs to be transferred between the microservices involved. Often, there will be a deep link for users to navigate between the apps.

Workflow integration can involve a business workflow that is executed across separate apps that also transfers data between the two apps. For a practical example, see [Example – Workflow Integration with Data Transfer](workflow-int-data-transfer).

### 7.1 Continuing the Workflow in Another App

To transfer a user from one app to the next in a business process, two options are available to use:

* Page URLs
* [Deep Link Module](https://appstore.home.mendix.com/link/app/43/) from the Mendix App Store

Both URLs and deep links can be used to continue the workflow in another app.

This table presents the pros and cons for these options:

| | Pros | Cons |
| --- | --- | --- |
| **Page URLs** | Built into the Desktop Modeler | Only for pages, no parameters possible    |
| **Deep link module** | Can start microflows; very flexible with link parameters | [Deep Link Module](https://appstore.home.mendix.com/link/app/43/) (with [Platform support](/developerportal/app-store/app-store-content-support)) |

Page URLs are very easy to use in the Mendix Platform, but the platform currently only supports opening a page and does not support custom parameters. More flexibility is needed for this case, both to trigger integration logic when opening a link and to link to specific objects using link parameters.

### 7.2 Copying Data Over Between Apps

When parts of data have been copied over, they need to be kept up to date. This is often done with a REST pull method (for more information, see the [REST Pull to Transfer Data](service-integration#pull-transfer) section of *Service Integration*).

To replicate data, use an API with a meaningful business object tree. Transferring related objects simultaneously is a little more difficult to set up, but it is better for data consistency. You should handle deleted data as “soft deletes” in the owning app so that the data will not disappear for the client and can be recovered if necessary.

In some cases, transactional data needs to be available instantly, and the app cannot afford to wait for an asynchronous pull process. When opening a deep link, the client app should synchronously trigger the existing pull process to retrieve the data on demand.

The Mendix Platform natively supports the following technologies for keeping data in sync between apps:

* REST
* SOAP
* OData

This table presents the pros and cons for these options:

| | Pros | Cons |
| --- | --- | --- |
| **REST** | Intended for publishing data; more efficient message format (JSON); reusable in custom widgets | Less support for data schema validation |
| **SOAP** | Strong schema support | Intended for operations in a verbose message format (XML) |
| **OData** | Using standard HTTP(S) connectivity; part of Mendix core | Does not support binary interface |
| **Batch** | N/A | N/A |
| **File** | N/A | N/A |
| **Database** | N/A | N/A |

The most frequently used option between Mendix apps is to use the REST protocol.

## 7 Summary

Process integration is one of the most important things an organization can focus on to increase automation, digitize processes, and make workflows leaner and more efficient. 

By using the Mendix Platform to build in a microservices-oriented way, process automation, case management, straight-through processing, and dashboards for exceptions can be adapted exactly to the problem that needs to be solved. This saves organizations large amounts in implementation and maintenance costs, and it makes the solution flexible for adapting to changing circumstances inside and outside the  organization.
