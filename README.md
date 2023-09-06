# Project Documentation/Report Outline

   - ### Title: Persistence Solution for User View History
   - ### Author(s): Collin Edwards
   - ### Date: July-Aug 2023

### Introduction
We have an count badge present for a particular item within our nav bar. To provide a better user experience, we want to implement a solution that collects, stores and later accesses user activity metadata to comparatively determine a count that represents new or unseen items.

### Requirements
- The solution will be wired into the preexisting API
- The solution will have a separate persistence layer to store the resulting data
- The solution will incorporate internal metrics for measuring the outcome/impact of the change
- The data stored within this layer will have a potential TTL of 6 months

### Design
Initially, I drafted an internal document (somewhat similar to this, but more robust) to outline the requirements, potential options, logic + data flow, as well as a work breakdown. After a number of discussions with my team, we settled on the persistence layer being a new DynamoDB table, and the data collection method being a kafka stream listener. It took a week or so to finalize the properties of the object that would populate the table. The benefit of a kafka stream as opposed to periodic data scraping (one of the proposed options) was the prospect of having realtime data related to the item count. 

### Implementation
As this is a backend project, the programming work was done entirely in Java, with DevOps/Data storage from AWS. 
To begin, I set up the DynamoDB table through a cloudformation script present on one of our existing APIs, as well as granting the necessary read/write permissions to the services that would need access. After the table was set up and verified, I created a service to handle posting and retrieving the data through instantiating a POJO. This allowed for ease of access between several different services that were required to construct or deconstruct the data. To actually populate the table, however, I had to tap into an internal kafka stream. This was accomplished through the use of a realtime stream listener, which collected events and filtered them based on our needs. I ended up creating this listener on one of our preexiting back-office processors. A separate service in the application was created to further validate the events, and subsequently post to the database. At this point, the data was being collected, but the API that handles count calculation was not connected. In order to begin sending the new count, I added logic to grab the events based on their unique IDs, and compare them to what the API was grabbing normally in order to determine which ones a given user has viewed already. The final step in this implementation was to alter the API contract to include the new count as part of the standard response, which was quick and easy.
### Testing
For each service, I wrote additional unit tests (with mocked data), as well as contract tests in order to verify the application as a whole was functioning correctly. 
### Maintenance
Currently, the ongoing maintenance includes custom metric tracking to narrow down areas for improvement, as well as latency and APM metrics to ensure everything is running smoothly after these changes.
### Outcomes
Through this project, I learned a lot about developing robust backend services for distributed systems at an enterprise level--most importantly, the level of interconnectivity and dependence that needs to be accounted for when adding a feature of this nature. A separate, unintended outcome this project had was that it allowed my team to collect previously unattainable metrics, increasing general observability and our capability to catch issues before they become a larger problem. Overall, I had a lot of fun working on this feature, and am excited to put my learnings toward more complex projects.
