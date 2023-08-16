## Storage > Block Storage > Release Notes

### August 29, 2023

* Added a high-performance block storage type (Performance Optimized SSD).
* Changed words such as volume and disk to **Block Storage**.

### May 30, 2023

* Added the encrypted block storage type.

### March 28, 2023

* Changed API endpoint

### September 27, 2022

#### Feature Updates

* Added a process to check snapshot integrity when running **Create Block Storage from a Snapshot**.

### March 29, 2022

#### Added Features

* Added the inter-region replication feature.

## May 26, 2020

* Released Public API v2, which is compatible with Openstack API. 

### March 24, 2020

#### Feature Updates

* Increased the maximum size of block storage creation from 1TB to 2TB.

### November 26, 2019

#### Added Features

* Default disk is displayed on the list of block storages.


### January 29, 2019

#### Added Features 

* Updated not to select snapshots as the origin, when a block storage is created
* Updated to choose availability zone and block type, when a block storage is created with snapshots


### September 21, 2017

#### Added Features

* Added Public APIs 
    * Block Storage, as well as Object Storage, could be enabled via APIs.  
    * Only limited features are available at the moment, but more features are to be included by adding more APIs in the near future.  
    * See [API Guide for Block Storage](/Storage/Block%20Storage/en/api-guide/).



### July 20, 2017

#### Bug Fixes 

* Fixed infrequent bugs in which creating a block storage was incomplete.  



### January 19, 2017

#### Feature Updates 

* Specify for the management of block storage attachement that a block storage could be attached to instances only when they belong to a same zone. 

#### Bug Fixes 

* Fixed infrequent failures of creating a block storage with snapshots. 

