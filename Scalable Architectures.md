## 1. Learning outcome
Besides functionality, you develop the architecture of enterprise software based on quality attributes. You especially consider attributes most relevant to enterprise contexts with high volume data and events. You design your architecture with future adaptation in mind. Your development environment supports this by being able to independently deploy and monitor the running parts of your application.

## 2. Non-Functionals
Unlike Functional requirements, Non-functional Requirements (NFR) focus mostly on the operation of a system rather than specific behaviour or functionalities. Accounting for implementation of NFR's need t be done during the system architecture design phase because of the architectural significance. My NFR's are as follows:

- **Speed**: All requests are handled within 2 seconds
- **Security**: All requests are secured using SSL
- **Security**: Passes quality gate in Sonarclould
- **Security**: Tested by Acunetix and issues mitigated
- **Reliability**: 96% uptime, 2.5% unexpected downtime, 1.5% expected downtime
- **Closed Source**: The project will not be freely available on the version control service
- **Data Integrity**: Personal data will be stored according to GDPR regulations

## 2. Architecture
As mentioned before, designing a well thought out architecture is essential to make sure NFR's can be realised. Therefore the following design has been created:

![Untitled Diagram drawio](https://user-images.githubusercontent.com/46562627/190365632-2873bd85-dd6a-4336-8fa1-f2bbb12972a2.png)


## 3. Messaging

## 4. Monitoring
