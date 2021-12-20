# PRP Kubernetes cluster usage workflow

## Background
The Cloud Computing Lab at the Department of Computer and Information Sciences, University of Delaware will be using computing resources provided by the National Research Platform (NRP) for conducting custom large-scale experiments on various research problems. As such, this document provides an overview of the technologies we need to know and learn in order to use those resources. I have created this document in a tutorial style so that all the necessary steps to successfully deploy and run your custom code in the NRP clusters are clearly established.

This document will also serve as our collective guide to deploying diverse applications, services, batch jobs, etc. to the NRP clusters. As we continue using these resources, all the contributors are expected to update the document with relevant information so that future members of the lab or other readers have access to precise instructions.

## People
* Namespace administrator (maintainer): Dixit Bhatta
* Namespace users (current): Ahmad Almansoor, Erfan Farhangi Maleki, Weibin Ma
* Namespace users (former): Sahar Nilipour Tabatabaee, Selim Emre Altıngöz
* Advisor (PI): Dr. Lena Mashayekhy

## Getting Started
We will get started with the initial information on the Nautilus cluster deployed under Pacific Research Platform (PRP). Please visit this link to read more about [PRP]( http://prp.ucsd.edu:8080/prp)

A namespace for our lab must have been already created in the PRP Kubernetes cluster. Check with the namespace administrator to get the namespace.

Please check your access to the PRP platform (subset of NRP) by logging** in at [Nautilus](https://nautilus.optiputer.net/) 
Note**: choose University of Delaware as account type when logging in, be careful not to choose Google.

After logging in, make sure you can see the namespace provided by your namespace admin under the "Namespaces overview" tab. 

The namespace admin can add you as a user of the namespace after you have logged in successfully in that system. Let the admin know as soon as you have successfully logged in.




