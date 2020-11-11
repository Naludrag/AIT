## AIT Lab 03 - Load balancing

**Author:** Müller Robin, Stéphane Teixeira Carvalho, Massaoudi Walid  
**Date:** 2020-10-07

### Introduction

### Task 1
#### 1.1
When we open the browser the application create a cookie for the user with the server s1 for example.

<img alt="Test 1" src="./imgRapport/1.1_1.PNG" width="700" >

In this screenshot we can see that the session ID NODESESSID is created.
This token is created by the web apps.

When we refresh the page we can see that the NODESESSID changed.
<img alt="Test 1" src="./imgRapport/1.1_2.PNG" width="700" >

This is the case because we changed of web app server, this time it is the web app s2, and so the session created before is unknown to the current server and a new one is created.

#### 1.2
It should keep the same session id even if the user refresh the page. And for that we should not speak with a different server but with the same that created the session.

#### 1.3
TODO : graphe

#### 1.4
<img alt="Test 1" src="./imgRapport/1.3.PNG" width="700" >

From the JMeter report we can clearly see that the server is changed on every new request so that means that a round robin is surely implemented.
#### 1.5
First here is the result from the JMeter.
<img alt="Test 1" src="./imgRapport/1.5.PNG" width="700" >

TDOD : Graphe

In this case all the requests were sent to the s2 server. We can also see a different comportement from the session handling.
<img alt="Test 1" src="./imgRapport/1.5_1.PNG" width="700" >
<img alt="Test 1" src="./imgRapport/1.5_2.PNG" width="700" >

This time because we comuncate with the same server we did not change of session id and the sessionView variable incremented as expected.
### Tâche 2

### Tâche 3

### Tâche 4

### Tâche 6

### Conclusion
