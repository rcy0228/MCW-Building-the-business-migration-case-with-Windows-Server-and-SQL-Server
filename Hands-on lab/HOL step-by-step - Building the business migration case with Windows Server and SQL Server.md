![](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/main/Media/ms-cloud-workshop.png "Microsoft Cloud Workshops")

<div class="MCWHeader1">
Building the business migration case with Windows Server and SQL Server
</div>

<div class="MCWHeader2">
Hands-on lab step-by-step
</div>

<div class="MCWHeader3">
October 2022
</div>


Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2022 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents** 

<!-- TOC -->

- [Building the business migration case with Windows Server and SQL Server hands-on lab step-by-step](#building-the-business-migration-case-with-windows-server-and-sql-server-hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Overview](#overview)
  - [Solution architecture](#solution-architecture)
  - [Requirements](#requirements)
  - [Before the hands-on lab](#before-the-hands-on-lab)
  - [Exercise 1: SQL database migration](#exercise-1-sql-database-migration)
    - [Task 1: Review Creation of Azure SQL MI **(Read-Only)** ](#task-1-review-creation-of-azure-sql-mi-read-only)
    - [Task 2: Install Data Migration Assistant](#task-2-install-data-migration-assistant)
    - [Task 3: Assess on-premises database compatibility](#task-3-assess-on-premises-database-compatibility)
    - [Task 4: Backup on-premises SQL database](#task-4-backup-on-premises-sql-database)
    - [Task 5: Migrate database to Azure SQL MI](#task-5-migrate-database-to-azure-sql-mi)
  - [Exercise 2: Create VM to migrate web application](#exercise-2-create-vm-to-migrate-web-application)
    - [Task 1: Create Windows Server 2022 Azure Edition VM for application hosting](#task-1-create-windows-server-2022-azure-edition-vm-for-application-hosting)
    - [Task 2: Check remote desktop access](#task-2-check-remote-desktop-access)
  - [Exercise 3: Azure Arc-enable on-premises VM](#exercise-3-azure-arc-enable-on-premises-vm)
    - [Task 1: Generate Azure Arc script to add server](#task-1-generate-azure-arc-script-to-add-server)
    - [Task 2: Run script to add server to Azure Arc](#task-2-run-script-to-add-server-to-azure-arc)
    - [Task 3: Verify Azure Arc-enabled VM](#task-3-verify-azure-arc-enabled-vm)
 
<!-- /TOC -->

# Building the business migration case with Windows Server and SQL Server hands-on lab step-by-step

## Abstract and learning objectives

In this hands-on lab, you will perform steps to migrate Windows Server and SQL Server workloads to Azure. You will go through provisioning a Windows Server VM, migrating a SQL Server database to Azure SQL Managed Instance (SQL MI), and Azure Arc-enable an on-premises Windows Server VM.

## Overview

In this lab, attendees will perform steps toward migrating Tailspin Toy's on-premises Windows Server and SQL Server workloads to Azure. Tailspin needs a new Windows Server VM created in Azure for hosting their Web application, an on-premises SQL Server database migrated to Azure SQL Managed Instance, and an on-premises Windows Server VM to be Azure Arc-enabled.

Tailspin already has a Hub and Spoke network setup in Azure with Azure Bastion for enabling remote management of Azure VM using Azure Bastion. The Azure resources provisioned throughout this lab will be deployed into this environment.

At the end of this hands-on lab, you will be better able to set up a Windows Server for application migration to Azure, migrate an on-premises SQL Database to Azure SQL Managed Instance, and Azure Arc-enable an on-premises virtual machine so it can be managed from Azure.

## Solution architecture

![Diagram showing on-premises network connected to Azure using Azure ExpressRoute with a Hub and Spoke network in Azure. The Spoke VNet contains the migrated Front-end, Back-end, and SQL Database workloads running within Subnets inside the Spoke VNet in Azure.](images/PreferredSolutionDiagram.png "Preferred Solution Diagram")

The diagram shows an on-premise network connected to Azure using Azure ExpressRoute with a Hub and Spoke network in Azure. The Spoke VNet contains the migrated Front-end, Back-end, and SQL Database workloads running within Subnets inside the Spoke VNet in Azure.


