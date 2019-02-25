[![MBARI logo](assets/images/logo-mbari-3b.png)](http://www.mbari.org)

# MBARI Media Management (M3)

MBARI's Media Management system (M3) is an evolution of MBARI's [Video Annotation and Reference System (VARS)](https://hohonuuli.github.io/vars/). It is specifically focused on managing digital video collections and annotations for scientific purposes.

## Overview

[View all GitHub repositories](https://github.com/mbari-media-management)

### Microservices

M3 is based around a collection of [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) [microservices](https://martinfowler.com/articles/microservices.html). MBARI has developed  microservices for:

- [Video asset management](https://github.com/mbari-media-management/vampire-squid)
- [Annotation managment](https://github.com/mbari-media-management/annosaurus)
- [Knowledgebase](https://github.com/mbari-media-management/vars-kb-server) for constraining annotations.
- [User managment](https://github.com/mbari-media-management/vars-user-server) to track annotators.
- [Image management](https://github.com/mbari-media-management/panoptes)

These microservices are each independant and can be mixed and matched for your own video applications. For development and simple deployments follow our [microservice guide](MICROSERVICES). 


### Applications

M3 includes several applications that are being developed to support video annotations. These include:

- [vars-annotation](https://github.com/mbari-media-management/vars-annotation) - A video annotation application
- [vars-kb](https://github.com/mbari-media-management/vars-kb) - An applicatino for managing the knowledgebase data.
- [Sharktopoda](https://github.com/mbari-media-management/Sharktopoda) - A macOS video player that integrates with vars-annotation.

### Libraries

To support our application development, MBARI has created the following [Java-based](https://www.java.com) libraries:

- [vcr4j](https://github.com/mbari-media-management/vcr4j) - A Java API for communicating with video 'devices'
- [vars-avfoundation](https://github.com/mbari-media-management/vars-avfoundation) - A Java API for native image capture on macOS.

## Workshops

- [Presentation at IMDIS 2018](https://youtu.be/W5s8tNlZWhE)
- [Big Ocean, Big Data Workshop](BOBD.md)

## Diagrams

[![Overview Diagram](assets/images/M3SimpleDeployment.jpg)](files/M3SimpleDeployment.pdf)

If this diagram causes a `WTF?` moment, you may want to read '[The non-techie's guide to servers](https://hackernoon.com/the-non-techies-guide-to-servers-af1fa3dbf7d8)'

## Acknowledgements

This project is funded by:

- [Monterey Bay Aquarium Research Institute](https://www.mbari.org/)
- [The David and Lucile Packard Foundation](https://www.packard.org/)

A big, hearty thanks to the following companies for providing licenses for their products. Their support of open-source and non-profit software development is extremely valuable and greatly appreciated.

- [AquaFold](https://www.aquafold.com/)
- [JProfiler](https://www.ej-technologies.com/products/jprofiler/overview.html)