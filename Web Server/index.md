# [Web server](https://en.wikipedia.org/wiki/Web_server)

A web server is a computer system that processes requests via HTTP, the basic network protocol used to distribute information on the World Wide Web. The term can refer to entire system, or specifically to the software that accepts and supervises the HTTP requests.

## Overview

The primary function of a web server is to store, process and deliver web pages to clients. The communication between client and server takes place using the Hypertext Transfer Protocol. Pages delivered are most frequently HTML documents, which may include images, style sheets and scripts in addition to text content.

A user agent, commonly a web browser or web crawler, initiates communication by making a request for a specific resource using HTTP and the server responds with the content of that resource or an error meesage if unable to do so. The resource is typically a real file on the server's secondary storage, but this is not necessarily the case and depends on how the web server is implemented.

While the primary function is to server content, a full implementation of HTTP also includes ways of receiving content from clients. This feature is used for submitting web forms, including uploading of files.

Many generic web servers also support server-side scipring using Active Server Pages, PHP, or other scritpting languages. This means that the behaviour of the web server can be scripted in separate files, while the actual server software remains unchanged. Usually, this function is used to generate HTML documents dynamically as opposed to returning static documents. The former is primarily used for retrieving or modifying information from databases. The latter is typucally much faster adn more easily cached but cannot deliver dynamic content.

Web servers are not only used for serving the World Wide Web. They can also be found embedded in devices such as printers, routers, webcams and serving only a local network. The web server may then be used as a part of a system for monitoring or administering the device in question. This usually means that no additional software has to be installed on the client computer, since only a web browser is required.

# [Application Server](https://en.wikipedia.org/wiki/Application_server)

An application server is a software framework that provides both facilities to create web applications and a server environment to run them. 

Application Server Frameworks contain a comprehensive service layer model. An application server acts as a set of components accessible to the software developer through a standard API defined for the platform itself. For Web applications, these components are usually performed in the same running environment as their web server, and their main job is to support the construction of dynamic pages. However, many application servers target much more than just Web page generation: they implement services like clustering, fail-over, and load-balacing, so developers can focus on implementing the business logic.

In the case of Java application servers, the server behaves like an extended virtual machine for running applications, transparently handling connections to the database on one side, and often, connections to the Web client on the other.

Other uses of the term may refer to ther services that a server makes available or the computer hardware on which the services run.

## Application Server definition

Application servers are system software upon which web application or desktop applications run. Application servers consist of web server connetors, computer programming languages, runtime libraries, database connectors, and the administration code needed to deploy, configure, manage, and connect these components on a web host. An application server runs behind a web Server in front of an SQL database. Web applications are computer code which run atop application servers and are written in the language the application server supports and call the runtime libraries and components the application server offers.

Many application servers exist. The choice impacts the cost, performance, reliability, scalability, and maintainability of a web application.

Proprietary application servers provide system services in a well-defined but proprietary manner. The application developers develop programs according to the specification of the application server. Dependence on a particular vendor is the drawback of this approach.

An opposite but analogous case is the Java EE platform. Java EE application servers provide system services in a well-defined, open, industry standard. The application developers develop programs according to the Java EE specification and not according to the application server. A Java EE application developed according to Java EE standard can be deployed in any Java EE application server making it vendor independent.

# [Database server](https://en.wikipedia.org/wiki/Database_server)

A database server is a computer program that provides database services to other computer programs or to computers, as defined by the client-server model. The term may also refer to a computer dedicated to running such a program. Database management systems frequently provide database-server functionality, and some database management systems rely exclusively on the client-server model for database access.

Users access a database server either through a "front end" running on the user's computer - which displays dequested data - or through the "bacl end", which runs on the server and handles tasks such as data analysis and storage.