<a id="storage-block-storage-release-notes"></a>
## Storage > Block Storage > Release Notes { #storage-block-storage-release-notes }

<a id="may-27-2026"></a>
## May 27, 2026 { #may-27-2026 }

<a id="feature-updates"></a>
### Feature Updates { #feature-updates }

* Improved block storage connection information display
    * You can now check which instance a block storage is connected to, even for terminated instances.
    * The existing **Connection Information** column has been renamed to **Connected Instance**.
* The default value of the limit parameter for the block storage and snapshot list retrieval API was adjusted to 100 and the maximum value to 1,000.
* Separated the block storage **Manage Connections** button into **Add Connection** and **Disconnect** buttons
    * The existing **Manage Connections** button was separated into **Add Connection** and **Disconnect** buttons.

<a id="may-27-2025"></a>
## May 27, 2025 { #may-27-2025 }

<a id="added-features"></a>
### Added Features { #added-features }

* Added the feature to move block storage
* Added the feature to set deletion policy when attaching block storage

<a id="march-4-2025"></a>
## March 4, 2025 { #march-4-2025 }

<a id="march-4-2025-added-features"></a>
### Added Features { #march-4-2025-added-features }

* Added quotas to limit the maximum number of snapshots per block storage

<a id="august-27-2024"></a>
## August 27, 2024 { #august-27-2024 }

<a id="august-27-2024-added-features"></a>
### Added Features { #august-27-2024-added-features }

* Added a feature to replicate block storage in the same region as a target region
* Added a feature to replicate block storage in antoher project to which you belong

<a id="may-28-2024"></a>
## May 28, 2024 { #may-28-2024 }

<a id="may-28-2024-added-features"></a>
### Added Features { #may-28-2024-added-features }

* You can create a snapshot for encrypted block storage.
* You can perform cross-region replication of encrypted block storage.
* Added the feature to change the block storage size.

<a id="april-23-2024"></a>
## April 23, 2024 { #april-23-2024 }

<a id="april-23-2024-feature-updates"></a>
### Feature Updates { #april-23-2024-feature-updates }

* Ended the use of u2-only storage type in Korea (Pangyo) region.
  
<a id="august-29-2023"></a>
## August 29, 2023 { #august-29-2023 }

<a id="august-29-2023-feature-updates"></a>
### Feature Updates { #august-29-2023-feature-updates }

* Changed Terms
    * Changed words such as volume and disk to **block storage**.

<a id="may-30-2023"></a>
## May 30, 2023 { #may-30-2023 }

<a id="may-30-2023-added-features"></a>
### Added Features { #may-30-2023-added-features }

* Added the encrypted block storage type.

<a id="march-28-2023"></a>
## March 28, 2023 { #march-28-2023 }

<a id="march-28-2023-feature-updates"></a>
### Feature Updates { #march-28-2023-feature-updates }

* Changed API endpoint

<a id="september-27-2022"></a>
## September 27, 2022 { #september-27-2022 }

<a id="september-27-2022-feature-updates"></a>
### Feature Updates { #september-27-2022-feature-updates }

* Added a process to check snapshot integrity when running **Create Block Storage from a Snapshot**.

<a id="march-29-2022"></a>
## March 29, 2022 { #march-29-2022 }

<a id="march-29-2022-added-features"></a>
### Added Features { #march-29-2022-added-features }

* Added the inter-region replication feature.

<a id="may-26-2020"></a>
## May 26, 2020 { #may-26-2020 }

<a id="may-26-2020-added-features"></a>
### Added Features { #may-26-2020-added-features }

* Released Public API v2
    * Released Public API v2, which is compatible with Openstack API. 

<a id="march-24-2020"></a>
## March 24, 2020 { #march-24-2020 }

<a id="march-24-2020-feature-updates"></a>
### Feature Updates { #march-24-2020-feature-updates }

* Increased the maximum size of block storage creation from 1TB to 2TB.

<a id="november-26-2019"></a>
## November 26, 2019 { #november-26-2019 }

<a id="november-26-2019-feature-updates"></a>
### Feature Updates { #november-26-2019-feature-updates }

* Default disk is displayed on the list of block storages.

<a id="january-29-2019"></a>
## January 29, 2019 { #january-29-2019 }

<a id="january-29-2019-added-features"></a>
### Added Features { #january-29-2019-added-features }

* Updated not to select snapshots as the origin, when a block storage is created
* Updated to choose availability zone and block type, when a block storage is created with snapshots

<a id="september-21-2017"></a>
## September 21, 2017 { #september-21-2017 }

<a id="september-21-2017-added-features"></a>
### Added Features { #september-21-2017-added-features }

* Added Public APIs 
    * Block Storage, as well as Object Storage, could be enabled via APIs.  
    * Only limited features are available at the moment, but more features are to be included by adding more APIs in the near future.  
    * See [API Guide for Block Storage](/Storage/Block%20Storage/en/public-api/).

<a id="july-20-2017"></a>
## July 20, 2017 { #july-20-2017 }

<a id="bug-fixes"></a>
### Bug Fixes { #bug-fixes }

* Fixed infrequent bugs in which creating a block storage was incomplete.  

<a id="january-19-2017"></a>
## January 19, 2017 { #january-19-2017 }

<a id="january-19-2017-feature-updates"></a>
### Feature Updates { #january-19-2017-feature-updates }

* Specify for the management of block storage attachement that a block storage could be attached to instances only when they belong to a same zone. 

<a id="january-19-2017-bug-fixes"></a>
### Bug Fixes { #january-19-2017-bug-fixes }

* Fixed infrequent failures of creating a block storage with snapshots. 
