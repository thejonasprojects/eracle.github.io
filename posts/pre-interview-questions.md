<!--
.. title:  Sr. backend software engineer - <br> interview questions:
.. slug: pre-interview-questions
.. date: 2017-10-10 09:05:10 UTC+02:00
.. tags: Amazon AWS, django, Interview, REST API, DRF, Mailgun
.. category: REST API
.. link: 
.. description: 
.. type: text
-->


I want to share here some answers I have written, during a pre-interview that I've approached for the role of sr. backend software engineer:

#### How would you design an horizontally scalable REST-API? Which techniques? Which Backend Architecture? Database?
Also influenced by the tools I've already used, I would use Amazon AWS, in particular, for the Django web server: Elastic Beanstalk and its Auto Scaling feature, which automatically deploys new back-end instances during peak load periods. That would implement an horizontally scalable REST-API.
Regarding the DB, horizontal scaling means using a solution with a read-write instance and a pool of read-only replicas. As this [Amazon RDS] article points out, this solution can be easily implemented by using Amazon RDS, which allows to configure read-only database replicas continuously kept up-to-date, under the hood, in an asynchronous way by the service it selves. Regarding the single read-write instance, Amazon RDS allows Vertical Scaling by offering DB instance classes with different performances and costs, so the right DB class can be chosen if the expected load of write queries is forecastable. 

#### How does Django supports backend scaling?
Django provides the Database [Routers configuration], which allows to define different database endpoints and different routing rules for each of them. If its performances are not satisfactory, with AWS, there is the possibility to configure an [HAProxy layer] as an AWS OpsWorks Stacks Layer. In both cases, write queries would be routed to the read-write DB instance and read queries to the read-only replicas.


#### Please, give some details over CSRF protection provided by Django REST framework.  
CSRF is a security concern that affects browsers or apps that contain an embedded browser, like an Android WebView.
There are different use cases for the REST APIs consumption:

- Stand-alone clients like Android apps or Desktop applications:
In this case, django-rest-framework's TokenAuthentication would be used, which does not run the CSRF security checks;
- AJAX calls of a client browser:
In this case, django-rest-framework's SessionAuthentication is the one used. It does run the CSRF security checks, so there are some API's design rules that have to be followed:
First, the REST APIs must have all GETs endpoints side effect free, that means, those endpoints do not have to change the DB's data, in other words, they are read-only toward the DB.
Second, for all the non GETs calls, each client must send a custom HTTP header, namelly 'X-CSRFToken', with the value of the CSRF token cookie the backend provides after authentication.


#### How would you implement the execution of a long-running task (5 minutes) triggered by a POST message?
Obviously, is not possible to execute the long-running task during the request-response cycle. Http requests are neither designed nor suited for this kind of behavior, and a time-out is what would happen during such as an heretic implementation.

As the one I have used on the [rbroker] source code, python offers a plenty of asynchronous libraries, in particular, I used django_q, which allows to define asynchronous tasks, which are executed by threads (workers) of a different process. 

Asynchronous libraries use to use brokers, which are data storages where the information needed by the workers are kept and shared.
For instance, I have configured django_q to use as broker the same underling database used by the others django apps. This is not efficient on production environments, since by competing on accessing the same database it would slow down the rest of the system.
Among the others python asynchronous libraries, worth to be noted, there is celery, which I had to deal with during one of my past working experiences. It offers a variety of different brokers, such as Redis, an efficient key-value storage, and Amazon SQS, a broker as-a-service offered by Amazon AWS. 

#### How would you implement the send of an e-mail, through a known REST API, triggered by a POST message received, taking into account that the send may be not successful?

During one of my past working experiences, we had to send emails during the normal user's work-flow, for instance subscription confirms. The project code-base was using the [django mail module]. After some time, some emails were hidden in the spam buckets of our users' email service provider. After that, we decided to outsource the task to an external provider, at that time the decision was [mailgun]. It allowed to free send up to 10,000 emails every month, in an asynchronous way, by using a REST API. In particular, by leveraging the [django-anymail] library, which offers a unified python API toward different providers, such as: Mailgun, Mailjet, Postmark, SendGrid, SparkPost and others.
Regarding the question, I would use mailgun and its tracking functionality, which allows to track the status of the sending process. In particular, there is the possibility to send webhooks, which are endpoint's URLs where mailgun POSTs all the events regarding the submitted emails.
So, when a particular event regarding a single email is received, a decision can be taken, but in order to here suggest a proper solution 
 there must be taken into account other project constraints, such as: importance of the emails sent or which are the policies over wrong email addresses.



[Amazon RDS]:				https://aws.amazon.com/blogs/database/scaling-your-amazon-rds-instance-vertically-and-horizontally/
[Routers configuration]:	https://docs.djangoproject.com/en/1.11/topics/db/multi-db/#an-example
[HAProxy layer]:			http://docs.aws.amazon.com/opsworks/latest/userguide/layers-haproxy.html
[rbroker]:					https://github.com/eracle/rbroker
[django mail module]:		https://docs.djangoproject.com/en/1.11/topics/email/
[mailgun]:					https://www.mailgun.com/
[django-anymail]:			https://github.com/anymail/django-anymail
[tracking functionalities]:	http://mailgun-documentation.readthedocs.io/en/latest/quickstart-events.html

