<!--
.. title:  Scraper on AWS Elastic Beanstalk - Please don't do that!
.. slug: scraper-on-aws-elastic-beanstalk-dont-do-that
.. date: 2017-09-09 09:05:10 UTC+02:00
.. tags: Amazon AWS, scrapy, django
.. category: DevOps
.. link: 
.. description: 
.. type: text
-->

![Amazon AWS - Autoscaling](http://docs.aws.amazon.com/autoscaling/latest/userguide/images/as-basic-diagram.png)

During one of my past working experiences, I had to modify and maintain an already started Django project.


The project architecture was designed to have a Django based REST APIs (with django-rest-framework) which used to run on Amazon Elastic Beanstalk. By not deepening too much on details, there was the core content of the database that was filled by a semi-automatic procedure. That procedure resided on the same project's code base, and involved the instantiation of a scraper module, which therefore, used to run on the same EC2 instance where the web server was. By designing it so, who developed it, had direct access to the database by using the Django's data access functionalities.
The issued scraper module was a [django-admin command] based on the python [requests] library, that used to parse the downloaded HTML files through the [lxml] library.


This architecture had at least two inconveniences: 


- First, since the EC2 instances were not designed to have an high number of requests per second, they were not configured to have a big amount of RAM memory. So, without manually changing the EB configuration, after being activated, the scraping process was killed before reaching complete usage of its available memory by the [watchdog] daemon of Ubuntu 16.04. 
In order to solve this problem, I had to manually change the EB configuration, by asking for a different instance type, for instance: t2.medium or t2.large.
This manual process creates a new EC2 instance with enhanced performances, and introduced a downtime period for the back-end. 


- Second, when the scraper run, slowed down the Django back-end, whom responses took longer time to be generated.
At that time the default configuration on the production environment had the Amazon Auto-scaling feature turned on, that feature, by continuously testing the response times of the instances, starts to create additional EC2 instances if the average response time raises too much. 
That used to happen multiple times during the execution of the Scraping process, since it was manually started on the same EC2 instance, and used to slow down the web server.
Let's naivelly close an eye on the fact that it was inefficient in terms of bills, the problem was that the Auto-scaling system after having increased the number of EC2 instances and having assessed that then the response time lowered, started the process of terminating some of them, in order to decrease their number, since it believes they are not needed anymore.
That termination used to happen in a random fashion, in other words, the terminated EC2 instances were randomly chosen, so, the first EC2 instance risked to be shutdown and to not finish the scraping process. 
As a possible solution, if the Amazon Auto-scaling feature would have been turned off, since on Amazon AWS the HTTPS configuration is strongly tightened with the Auto-scaling one, the production platform turned out to be not reachable via its normal URL, in other words: it was out of service for users.



After some troubles dealing with this architecture, I decided to solve the problem and migrate the scraper module to a different code base that leverages the [scrapy] library and [scrapinghub] platform, but this is another (interesting) story.

### EDIT:
I just get aware that I could have configured autoscaling for [instance protection] to avoid the termination of the issued instance where the scraper ran. Well, in the end, by having moved the scraper out of the web server, and having modified the project toward a microservices architecture, in my opinion, still, was not a bad idea.


[watchdog]:	http://manpages.ubuntu.com/manpages/xenial/man8/watchdog.8.html
[scrapy]:		https://scrapy.org/
[scrapinghub]:	https://scrapinghub.com/
[requests]:		http://docs.python-requests.org/en/master/
[lxml]:			https://packages.ubuntu.com/xenial/python-lxml
[django-admin command]:	https://docs.djangoproject.com/en/1.11/howto/custom-management-commands/
[instance protection]:	http://docs.aws.amazon.com/autoscaling/latest/userguide/as-instance-termination.html#instance-protection-instance
