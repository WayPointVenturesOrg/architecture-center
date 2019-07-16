---
title: "Azure readiness storage design guidance"
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: Azure readiness storage design guidance
author: BrianBlanchard
ms.author: brblanch
ms.date: 05/15/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
---

# Storage design decisions

Storage capabilities are critical to supporting workloads and services being hosted in the cloud. As part of your cloud adoption readiness preparations, review this article to help plan for and address your storage needs using Azure Storage services.

## Select the storage tools and services to support your workloads

[Azure Storage](/azure/storage) is the Azure platform's managed service for providing cloud storage that is highly available, secure, durable, scalable, and redundant. It is composed of several core services and supporting features. Review the scenarios and considerations discussed below to choose the relevant Azure services and the correct architectures to fit your organization's workload, governance, and data storage requirements.

![Azure storage decision tree](../../_images/ready/storage-decision-tree.png)

### Key questions

- **Will your workload require disk storage to support the deployment of IaaS virtual machines?** [Azure Disk Storage](/azure/virtual-machines/windows/managed-disks-overview) is used to provide virtual disk capabilities for IaaS virtual machines. 
- **Will you need to provide downloadable images, documents, or other media as part of your workload?** [Azure Blob Storage](/azure/storage/blobs/storage-blobs-introduction) provides the ability to [host static files](/azure/storage/blobs/storage-blob-static-website), which are then accessible for download over the Internet. Assets hosted in Blob Storage can be made public, or [limited to users authorized](/azure/storage/common/storage-auth) via Azure Active Directory, shared keys, or shared access signatures.
- **Will you need a location to store virtual machine or application logs and analytics data?** Azure Blob Storage can also be used to [store Azure Monitor log data](/azure/storage/common/storage-analytics).
- **Will you need to provide a location for backup, disaster recover, or archiving workload-related data?** Azure Blob Storage is used by Azure Disk Storage to provide [backup and disaster recovery capabilities](/azure/virtual-machines/windows/backup-and-disaster-recovery-for-azure-iaas-disks). Blob storage can also be used as a location to back up other resources, such as on-premises or IaaS VM-hosted [SQL Server data](https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service?view=sql-server-2017).
- **Will you need to support big data analytics workloads?** Built on top of Azure Blob Storage, [Azure Data Lake Storage Gen 2](/azure/storage/blobs/data-lake-storage-introduction) is capable of supporting large enterprise data lake functionality, and can handle storing petabytes of information while sustaining hundreds of gigabits of throughput.
- **Do you need to provide cloud-native file shares?** Azure has two primary services providing cloud-hosted file shares. [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction) provides high-performance NFS shares well suited to common enterprise workloads such as SAP. [Azure Files](/azure/storage/files/storage-files-introduction) provides file shares accessible over SMB 3.0 and HTTPS.
- **Will you need to support hybrid cloud storage for on-premises high-performance computing (HPC) workloads?** [Avere vFXT for Azure](/azure/avere-vfxt/avere-vfxt-overview) provides a hybrid caching solution allowing you to expand your on-premises storage capabilities using cloud-based storage. It is optimized for read-heavy HPC workloads involving compute farms of 1000 to 40,000 CPU cores, and can integrate with on-premises hardware NAS, Azure Blob storage, or both.
- **Do you need to perform large-scale archiving and syncing of your on-premises data to the cloud?** [Azure Data Box](/azure/databox-family/) products are designed to help move large amounts of data from your on-premises environment to the cloud. [Data Box Gateway](/azure/databox-online/data-box-gateway-overview) is a virtual device that resides on-premises that helps to manage large-scale data migration to the cloud. If you need to analyze, transform, or filter data before moving it to the cloud, [Data Box Edge](/azure/databox-online/data-box-edge-overview) is an AI-enabled physical edge computing device deployed to your on-premises environment to accelerate processing and secure transfer of data to Azure.
- **Do you want to expand an existing on-premises file share to take advantage of cloud storage?** [Azure File Sync](/azure/storage/files/storage-sync-files-deployment-guide) allows you to use the Azure Files service as an extension of files shares hosted on your on-premises Windows Server machines. This syncing service transforms Windows Server into a quick cache of your Azure file share, and allows your on-premises machines to access this share use any protocol that's available on Windows Server.

## Common storage scenarios

Azure offers multiple products and services that provide different storage capabilities. In addition to the decision tree listed above, the following tables provide a series of potential storage scenarios and the recommended Azure services to address that scenario's requirements.

### Block storage scenarios

<!-- markdownlint-disable MD033 -->

| **Scenario** | **Suggested&nbsp;Azure&nbsp;services** | **Considerations for suggested services** |
|---|---|---|
| I have bare metal servers or VMs (Hyper-V or VMware) with direct attached storage running LOB applications. | [Azure Disk Storage (premium SSD)](/azure/virtual-machines/windows/disks-types#premium-ssd) | For production services, premium SSDs provide consistent low-latency coupled with high IOPS and throughput. |
| I have servers to host web and mobile apps. | [Azure Disk Storage (standard SSD)](/azure/virtual-machines/windows/disks-types#standard-ssd) | Standard SSD IOPS and throughput might be sufficient (at a lower cost than premium SSDs) for CPU bound web/app servers in production. |
| I have an enterprise SAN or all-flash array (AFA). | [Azure Disk Storage (premium or ultra SSD)](/azure/virtual-machines/windows/disks-types) <br/><br/> [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction) | Ultra SSDs are NVMe-based, offer submillisecond latency with high IOPS and bandwidth, and are scalable up to 64 TiB. Choice of premium SSD versus ultra SSD depends on peak latency, IOPS, and scalability requirements. |
| I have HA clustered servers (such as SQL Server FCI or Windows Server failover clustering). | [Azure Files (premium)](/azure/storage/files/storage-files-planning#file-share-performance-tiers)<br/> [Azure Disk Storage (premium or ultra SSD)](/azure/virtual-machines/windows/disks-types) | Clustered workloads require multiple nodes to mount the same underlying shared storage for failover or high availability. Premium file shares offer shared storage mountable via SMB. Shared block storage can also be configured on premium or ultra SSDs using [partner solutions](https://azuremarketplace.microsoft.com/marketplace/apps/sios_datakeeper.sios-datakeeper-8?tab=Overview). |
| I have a relational database or data warehouse workload (such as SQL Server or Oracle). | [Azure Disk Storage (premium or ultra SSD)](/azure/virtual-machines/windows/disks-types) | Choice of premium SSD versus ultra SSD depends on peak latency, IOPS, and scalability requirements. Ultra SSDs also reduce complexity by removing the need for Storage Pool configuration for scalability (see [details](https://azure.microsoft.com/blog/mission-critical-performance-with-ultra-ssd-for-sql-server-on-azure-vm)). |
| I have a NoSQL cluster (such as Cassandra or MongoDB). | [Azure Disk Storage (premium SSD)](/azure/virtual-machines/windows/disks-types#premium-ssd) | Azure Disk Storage's premium SSD offering provides consistent low-latency coupled with high IOPS and throughput. |
| I am running containers with persistent volumes. | [Azure Files (standard or premium)](/azure/storage/files/storage-files-planning) <br/><br/> [Azure Disk Storage (standard, premium, or ultra SSD)](/azure/virtual-machines/windows/disks-types) | File (RWX) and block (RWO) volumes driver options available for both AKS and custom Kubernetes deployments. Persistent volumes (PVs) can map to either an Azure Disk Storage disk or managed Azure Files share. Choose premium versus standard options base on workload PV requirements. |
| I have a data lake (such as a Hadoop cluster for HDFS data). | [Azure Data Lake Storage Gen 2](/azure/storage/blobs/data-lake-storage-introduction) <br/><br/> [Azure Disk Storage (standard or premium SSD)](/azure/virtual-machines/windows/disks-types) | The Azure Data Lake Storage (ADLS) Gen 2 feature of Azure Blob Storage provides server-side HDFS compatibility and petabyte scale for parallel analytics along with HA and reliability. Software like Cloudera can also use premium or standard SSDs on master/worker nodes if needed. |
| I have an SAP or SAP HANA deployment. | [Azure Disk Storage (premium or ultra SSD)](/azure/virtual-machines/windows/disks-types) | Ultra SSDs are optimized to offer submillisecond latency for tier-1 SAP workloads. Ultra SSDs are now in preview. Premium SSDs coupled with M-Series offer a GA option. |
| I have a DR site with strict RPO/RTO that syncs from my primary servers. | [Page blobs](/azure/storage/blobs/storage-blob-pageblob-overview) | Page blobs are used by replication software to enable low-cost replication to Azure without the need for compute VMs until failover occurs. Details can be found in the [Azure Disk Storage documentation](/azure/virtual-machines/windows/backup-and-disaster-recovery-for-azure-iaas-disks). Note: Page blobs support a maximum of 8 TB. |

### File and object storage scenarios

| **Scenario** | **Suggested&nbsp;Azure&nbsp;Service(s)** | **Considerations for suggested services** |
|------------------------------------------------------------------------------------------|------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| I have a Windows File Server. | [Azure Files](/azure/storage/files/storage-files-planning) <br/><br/> [Azure File Sync](/azure/storage/files/storage-sync-files-planning) | With Azure File Sync, you can store rarely used data on cloud-based Azure file shares while caching your most frequently used files on-premises for fast, local access performance. You can also use multisite sync to keep files in sync across multiple servers. If you plan to migrate your workload to a cloud-only deployment, Azure Files may be sufficient. |
| I have an Enterprise NAS (such as NetApp Filers or Dell-EMC Isilon.) | [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction) <br/><br/> [Azure Files (premium)](/azure/storage/files/storage-files-planning#file-share-performance-tiers) | If you have an on-premises deployment of NetApp, consider Azure NetApp Files to migrate deployment to Azure. If you are using or migrating to a Windows or Linux server or have basic functionality needs from a file share, consider Azure Files; for continued on-premises access, use Azure File Sync to Azure file shares with on-premises using a cloud tiering mechanism. |
| I have a file share (SMB or NFS). | [Azure Files (standard or premium)](/azure/storage/files/storage-files-planning) <br/><br/> [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction) | Choice of premium versus standard Azure Files tiers depends on IOPS, throughput, and latency consistency needs. If you have an on-premises deployment of NetApp, consider Azure NetApp Files. If you need to migrate your ACLs and timestamps to the cloud, Azure File Sync can bring all of these settings to your Azure file shares as a convenient migration path. |
| I have an object storage system on premises for petabytes of data (such as Dell-EMC ECS). | [Azure Blob Storage](/azure/storage/blobs/storage-blobs-introduction) |  Azure Blob Storage provides premium, hot, cool, and archive tiers to match your workloads performance and cost needs. |
| I have a DFSR deployment or other way of handling branch offices. | [Azure Files](/azure/storage/files/storage-files-planning) <br/><br/> [Azure File Sync](/azure/storage/files/storage-sync-files-planning) | Azure File Sync offers multisite sync to keep files in sync across multiple servers and native Azure file shares in the cloud. Get to a fixed storage footprint on-premises by using cloud tiering, which transforms your server into a cache for the relevant files while scaling cold data in Azure file shares. |
| I have a tape library (on-premises or offsite) for Backup/DR or long-term data retention. | [Azure Blob Storage (Cool or Archive tiers)](/azure/storage/blobs/storage-blob-storage-tiers) |  Archive tier will have the lowest possible cost but can require hours to copy the offline data to a cool, hot, or premium tier of storage to allow access. Cool tier provides instantaneous access at low cost. |
| I have file or object storage configured to receive my backups. | [Azure Blob Storage (Cool or Archive tiers)](/azure/storage/blobs/storage-blob-storage-tiers) <br/>[Azure File Sync](/azure/storage/files/storage-sync-files-planning) | To back up data for long-term retention with lowest-cost storage, move data to Azure Blob Storage and use Cool and Archival tiers. To enable fast disaster recovery for file data on a server (on-premises or Azure VM), sync shares to individual Azure file shares via Azure File Sync. With Azure file share snapshots, you can restore previous versions and sync them back to connected servers or access them natively in the Azure file share. |
| I run data replication to a disaster recovery (DR) site. | [Azure Files](/azure/storage/files/storage-files-planning) <br/><br/> [Azure File Sync](/azure/storage/files/storage-sync-files-planning) | Azure File Sync removes the need for a DR server and stores files in native Azure SMB shares. Fast Disaster Recovery will rebuild any data on a failed on-premises server quickly. You can even keep multiple server locations in sync or use cloud tiering to store only relevant data on-premises. |
| I manage data transfer in disconnected scenarios. | [Azure Data Box Edge/Gateway](/azure/databox-online) | Using Data Box Edge/Gateway you can copy data in disconnected scenarios. When the gateway is offline it will save all files you copy in the cache, then upload when you’re connected. |
| I manage an ongoing data pipeline to the cloud | [Azure Data Box Edge/Gateway](/azure/databox-online) | Move data to the cloud from systems that are constantly generating data just by having them copy that data straight to the Storage Gateway. If they need to access that data later, it’s right there where they put it. |
| I have bursts of quantities of data that shows up all at once. | [Azure Data Box Edge/Gateway](/azure/databox-online) | Manage large quantities of data that show up all at once, like when an autonomous car pulls back into the garage, or a gene sequencing machine finishes its analysis. Copy all that data to the Azure Data Box Gateway at fast local speeds then let Gateway upload it as your network allows

### Planning based on data workloads

| **Scenario** | **Suggested&nbsp;Azure&nbsp;Service(s)** | **Considerations for suggested services** |
|---------------------------------------------------------------------------------------------------------------------------------|---------------------------------|-----------------------------------------------------------------------------------------------------------------|
| Developing a new cloud-native application that needs to persist unstructured data. | [Azure Blob Storage](/azure/storage/blobs/storage-blobs-introduction) | |
| Migrating data from an on-premises NetApp to Azure. | [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction) | |
| Migrating data from an on-premises Windows File Servers to Azure. | [Azure Files](/azure/storage/files/storage-files-planning) | |
| Move file data to the cloud but continue to primarily access the data from on-premises. | [Azure Files](/azure/storage/files/storage-files-planning) <br/><br/> [Azure File Sync](/azure/storage/files/storage-sync-files-planning) | |
| "Burst compute" - NFS/SMB read-heavy file-based workloads with data assets residing on-premises while computation runs in the cloud. | [Avere vFXT for Azure](/azure/avere-vfxt/avere-vfxt-overview) | IaaS scale-out NFS/SMB file caching |
| Move an on-premises application that uses local disk or iSCSI. | [Azure Disk Storage](/azure/virtual-machines/windows/managed-disks-overview) | |
| Migrate a container-based application with persistent volumes. | [Azure Disk Storage](/azure/virtual-machines/windows/managed-disks-overviews) <br/><br/> [Azure Files](/azure/storage/files/storage-files-planning) | |
| Move file shares to the cloud that are not Windows Server or NetApp. | [Azure Files](/azure/storage/files/storage-files-planning) <br/><br/> [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction) | Protocol Support Regional Availability Performance Requirements Snapshot and Clone Capabilities Price Sensitivity |
| Transfer terabytes to petabytes of data from on-premises to Azure. | [Data Box Edge](/azure/databox-online/data-box-edge-overview) | |
| Process data before transferring to Azure. | [Data Box Edge](/azure/databox-online/data-box-edge-overview) | |
| Continuous ingestion of data in an automated way with local cache. | [Data Box Gateway](/azure/databox-online/data-box-gateway-overview) | |

## Learn more about Azure storage services

After identifying the Azure tools that best match your requirements, use the detailed documentation linked below to familiarize yourself with these services.

| **Service**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | **Description** |
|-------------------------------------|------------|
| [Azure Blob Storage](/azure/storage/blobs/storage-blobs-introduction) | Azure Blob storage is Microsoft's object storage solution for the cloud. Blob storage is optimized for storing massive amounts of unstructured data. Unstructured data is data that does not adhere to a particular data model or definition, such as text or binary data.<br/><br/>Blob storage is designed for:<ul><li>Serving images or documents directly to a browser.</li><li>Storing files for distributed access.</li><li>Streaming video and audio.</li><li>Writing to log files.</li><li>Storing data for backup and restore, disaster recovery, and archiving.</li><li>Storing data for analysis by an on-premises or Azure-hosted service.</li></ul> |
| [Azure Data Lake Storage Gen 2](/azure/storage/blobs/data-lake-storage-introduction) | Blob storage supports Azure Data Lake Storage Gen2, Microsoft's enterprise big data analytics solution for the cloud. Azure Data Lake Storage Gen2 offers a hierarchical file system as well as the advantages of Blob storage, including low-cost, tiered storage; high availability; strong consistency; and disaster recovery capabilities. |
| [Azure Disk Storage](/azure/virtual-machines/windows/managed-disks-overview) | Azure Disk Storage offer persistent, high-performance block storage to power your Azure Virtual Machines. Azure disks are highly durable, secure, and offer the industry’s only single instance SLA for VMs using premium or ultra SSDs ([learn more about disk types](/azure/virtual-machines/windows/disks-types)). Azure disks provide high availability with Availability Sets and Availability Zones that map to your Azure Virtual Machines fault domains. In addition, Azure disks are managed as a top-level resource in Azure providing Azure Resource Manager capabilities like role-based access control (RBAC), policy, and tagging by default. |
| [Azure Files](/azure/storage/files/storage-files-planning) | Azure Files provides fully managed, native SMB file shares as a service&mdash;without the need to run a VM. You can mount an Azure Files share as a network drive to any Azure VM or on-premises machine. |
| [Azure File Sync](/azure/storage/files/storage-sync-files-planning) | Azure File Sync can be used to centralize your organization's file shares in Azure Files, while keeping the flexibility, performance, and compatibility of an on-premises file server. Azure File Sync transforms Windows Server into a quick cache of your Azure file share. |
| [Azure NetApp Files](/azure/azure-netapp-files/azure-netapp-files-introduction) | The Azure NetApp Files service is an enterprise-class, high-performance, metered file storage service. Azure NetApp Files supports any workload type and is highly available by default. You can select service and performance levels and set up snapshots through the service. |
| [Data Box Edge](/azure/databox-online/data-box-edge-overview) | Data Box Edge is an on-premises network device that moves data into and out of Azure and has AI-enabled edge compute to preprocess data during upload. Data Box Gateway is a virtual version of the device with the same data transfer capabilities. |
| [Data Box Gateway](/azure/databox-online/data-box-gateway-overview) | Azure Data Box Gateway is a storage solution that enables you to seamlessly send data to Azure. Data Box Gateway is a virtual device based on a virtual machine provisioned in your virtualized environment or hypervisor. The virtual device resides in your premises and you write data to it using the NFS and SMB protocols. The device then transfers your data to Azure block blob, page blob, or Azure Files. |
| [Avere vFXT for Azure](/azure/avere-vfxt/avere-vfxt-overview) | Avere vFXT for Azure is a filesystem caching solution for data-intensive high-performance computing (HPC) tasks. Take advantage of cloud computing's scalability to make your data accessible when and where it's needed&mdash;even for data that’s stored in your own on-premises hardware. |

## Data redundancy and availability

Azure Storage has various redundancy options that ensure durability and high availability based on customer needs: locally redundant storage (LRS), zone-redundant storage (ZRS), geo-redundant storage (GRS) and read-access geo-redundant storage (RA-GRS).

See [documentation on Azure Storage redundancy](/azure/storage/common/storage-redundancy) to learn more about these capabilities and how you can decide on the best redundancy option for your use-cases. Also, Service-Level Agreement (SLA) for storage services provide guarantees that are financially backed. For more information, see the [SLA for managed disks](https://azure.microsoft.com/support/legal/sla/managed-disks/v1_0), [SLA for virtual machines](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8), and [SLA for storage accounts](https://azure.microsoft.com/support/legal/sla/storage/v1_4).

Refer to the article [Backup and DR for Azure Disk Storage](/azure/virtual-machines/windows/backup-and-disaster-recovery-for-azure-iaas-disks) for guidance on planning the right solution for Azure disks.

## Security

To help you protect your data in the cloud, Azure Storage offers several best practices for data security and encryption for data at rest and in transit. You can:

- Secure the storage account by using role-based access control (RBAC) and Azure Active Directory (Azure AD).
- Secure data in transit between an application and Azure by using client-side encryption, HTTPS, or SMB 3.0.
- Set data to be automatically encrypted when it's written to Azure Storage by using Storage Service Encryption.
- Grant delegated access to the data objects in Azure Storage by using shared access signatures.
- Use analytics to track the authentication method that someone is using when they access Storage.

These security features apply to Azure blob storage (block and page) and Files. You can find detailed Storage security guidance in the [Azure Storage security guide](/azure/storage/common/storage-security-guide).

[Azure Storage Service Encryption](/azure/storage/storage-service-encryption) (SSE) provides encryption-at-rest and safeguards your data to meet your organizational security and compliance commitments. SSE is enabled by default for all managed disks, snapshots, and images in all the Azure regions. Starting June 10, 2017, all new managed disks, snapshots, Images and new data written to existing managed disks are automatically encrypted-at-rest with keys managed by Microsoft. Visit the [FAQ for managed disks](/azure/storage/storage-faq-for-disks#managed-disks-and-storage-service-encryption-sse) page for more details. SSE with customer-managed keys (CMK) stored in Azure Key Vault will be available in all the regions by the end of CY2019.

Azure Disk Encryption allows you to encrypt managed disks attached to IaaS VMs as OS and data disks at rest and in transit using your keys stored in [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault). For Windows, the drives are encrypted using industry-standard [BitLocker](/windows/security/information-protection/bitlocker/bitlocker-overview) encryption technology. For Linux, the disks are encrypted using the [dm-crypt](https://wikipedia.org/wiki/Dm-crypt) subsystem. The encryption process is integrated with Azure Key Vault to allow you to control and manage the disk encryption keys. For more information, see [Azure Disk Encryption for Windows and Linux IaaS VMs](/azure/security/azure-security-disk-encryption-overview).

## Regional availability

Azure allows you to deliver services at the scale you need to reach your customers and partners, *wherever they are*. The [managed disks](https://azure.microsoft.com/global-infrastructure/services/?products=managed-disks)  and [Azure Storage](https://azure.microsoft.com/global-infrastructure/services/?products=storage) pages shows which services are available in which regions today. Checking regional availability of the services beforehand will enable making the right decision for your workload and customer needs.

Managed disks are available in all Azure regions with premium SSD and standard SSD offerings. While ultra SSDs remain in public preview, they are only offered in a single availability zone of the East US 2 region. Verify the regional availability while planning mission-critical top-tier workloads that need ultra SSDs.

Similarly, Hot/Cool Blob Storage, Azure Data Lake Storage Gen2, and File Storage are available in all Azure regions. Archival Blob storage, premium file shares, and Premium Block Blob storage are limited to certain regions and we recommend referring to the regions page to check the latest status.

To learn more about Azure global infrastructure, you can visit the [Azure regions page](https://azure.microsoft.com/global-infrastructure/regions). You can also consult the [products available by region](https://azure.microsoft.com/global-infrastructure/services/?products=storage) page for specific details on what's available in each Azure region.

## Data residency and compliance requirements

Legal and contractual requirements related to data storage will often apply to your workloads. These requirements may vary based on the location of your organization, the jurisdiction where files and data are stored and processed, and your applicable business sector. Components of understanding data obligations include data classification, data location, and the respective responsibilities for data protection under the shared responsibility model. For help with understanding these requirements, see the white paper [Achieving Compliant Data Residency and Security with Azure](https://azure.microsoft.com/resources/achieving-compliant-data-residency-and-security-with-azure).

As part of these compliance efforts, you may need to control where your storage resources are physically located. Azure regions are organized into groups called geographies. An [Azure geography](https://azure.microsoft.com/global-infrastructure/geographies) ensures that data residency, sovereignty, compliance, and resiliency requirements are honored within geographical and political boundaries. If your workloads are subject to data sovereignty or other compliance requirements, storage resources must be deployed to regions in a compliant Azure geography.