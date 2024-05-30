# Route Approval Check script - version 2 ("Golden List")
## Important Note
Version 1 of the script is meant to be scheduled every day. The script checks the current results (from "today") and compares to the results generated "yesterday". 
The script prints the differences in the command line and sends a notification e-mail.
[Link to version 1](https://github.com/JakubD-AVX/aviatrix-route-approval-check-python-script/)
The limitation of version 1 is that it works only with "Connection Mode" Route Approval feature. 

Version 2 of the script leverages "golden list" files that store the desired list of approved CIDRs. The script (v2) compares the current results to the information taken from "golden list" files.
The script (v2) prints the differences in the command line and sends a notification e-mail.
What is important, it (v2) works for both "Gateway Mode" and "Connection Mode" Route Approval feature.

## Introduction
There is an Aviatrix feature called "BGP Route Approval" that could be enabled on Aviatrix Gateways ([Aviatrix Docs - BGP Route Approval](https://docs.aviatrix.com/documentation/latest/building-your-network/transit-bgp-route-approval.html?expand=true)).

The Route Approval feature maintains two lists of CIDRs: "Approved CIDR" list and "Pending CIDR" list:
- "Pending CIDR" is a list of CIDRs that remote BGP Peer advertises to us but we do not have them installed/accepted on Aviatrix Gateway yet,
- "Approved CIDR" is a list of CIDRs that remote BGP Peer advertises to us nad we have accepted/installed on Aviatrix Gateway.

The Route Approval feature could operate in two Modes (it must be set up per Gateway):
- "Gateway Mode", in this mode "Approved CIDR" and "Pending CIDR" lists are maintaned for all connections established by Gateway,
- "Connection Mode", in this mode "Approved CIDR" and "Pending CIDR" lists are maintened separately for each connection established by Gateway.

The purpose of the script is to validate whether the "Approved CIDR" list and the "Pending CIDR" list changed between current check and "golden list of CIDRs". 
The knowledge about whether those lists changed could be crucial if someone wants to monitor whether: 
- there are new CIDRs that have been approved, 
- there are some previously approved CIDRs that have been removed, 
- there are new pending CIDRs advertised from BGP Peer, 
- some pending CIDRs have been removed.

The "golden list CIDRs" are kept in **golden_list** folder. 
For Gateway that operates in "Connection Mode":
- there is 1 file required per each BGP connection and this file must be created before running the script:

  transit-gw-name_connection_connection-name_approved_cidr_list_golden_list.csv

  The purpose of the file is to keep a list of all approved CIDRs.
For Gateway that operates in "Gateway Mode":
- there is 1 file required per Gateway and this file must be created before running the script:

  transit-gw-name_Gateway-Mode_approved_cidr_list_golden_list.csv

  The purpose of the file is to keep a list of all approved CIDRs.
  
The script can be scheduled to be run every day or even multiple times a day. 
During the script execution the following 4 files are created in **temp_files** folder for each BGP connection (in case "Connection Mode" is used):
- transit-gw-name_connection_connection-name_approved_cidr_list_date_yyyy-mm-dd.csv
  The purpose of the file is to keep a list of all approved CIDRs.
- transit-gw-name_connection_connection-name_pending_cidr_list_date_yyyy-mm-dd.csv
  The purpose of the file is to keep a list of all pending CIDRs.
- transit-gw-name_connection_connection-name_total_approved_cidr_date_yyyy-mm-dd.csv
  The purpose of the file is to keep the total number of approved CIDRs.
- transit-gw-name_connection_connection-name_total_pending_cidr_date_yyyy-mm-dd.csv
  The purpose of the file is to keep the total number of approved CIDRs.

OR only 1 file is created in **temp_files** folder for the gateway (in case "Gateway Mode" is used):

- transit-gw-name_Gateway-Mode_total_pending_cidr_date_yyyy-mm-dd.csv
  The purpose of the file is to keep the total number of approved CIDRs.

![image](https://github.com/JakubD-AVX/aviatrix-route-approval-check-python-script-v2/assets/98452952/f16b18d3-796e-4816-b7d6-284fc58fd6b6)


## Requirements
Please keep in mind that script only works for **Aviatrix Transit Gateways** that have "BGP Route Approval" feature enabled in **Connection-Mode**.
The script uses .env file for keeping the information required to log in to the Aviatrix Controller (you can use read-only User Account) and to send the email notifications.
The following variables must be updated by you before executing the script.
The content of .env file:

```
# Aviatrix Controller
CONTROLLER_IP = "IP-of-your-Aviatrix-Controller"
CONTROLLER_USER = "your-Aviatrix-Controller-Username"
CONTROLLER_PASSWORD = "your-Aviatrix-Controller-Username-Password"

# Email details
SENDER_EMAIL = "your-sender-email-address"
EMAIL_PASSWORD = "your-email-application-password"
RECEIVER_EMAIL = "your-receiver-email-address"
SMTP_PORT = "smtp-port"          
SMTP_SERVER = "smtp-server-fqdn" 
```

## Usage Examples
Please be aware that you must provide the name of the Aviatrix Transit Gateway when executing the script.
Please also notice the "golden list" files must be created before running the script
Example:
```
$ python3 route_approval_check.py transit-90
```
The *transit-90* is a name of my Transit Gateway.

## Output
![image](https://github.com/JakubD-AVX/aviatrix-route-approval-check-python-script-v2/assets/98452952/8c19c2e7-6ec0-44dd-bdf1-faf07c84fd52)

Besides the files that are created by the script, the script also generates some outputs and sends notification e-mails.
The script generates the outputs that show the following pieces of information:
- name of the Aviatrix Transit Gateway that has been checked
- name of each connection
- for each connection: a full list of CIDRs that are present in the "golden list" but are not approved by Route Approval Feature
- for each connection: a full list of CIDRs that are approved by Route Approval Feature but are not present in the "golden list"

In case the list of CIDRs (either approved or pending) between current check and "golden list" is not equal -> script sends a notification e-mail(s).
### Executing the script with no "golden list" files
Once the script is executed and there are no "golden list" files the script prints a notificationand the script cannot compare the data gathered by current check against the "golden list".
However, the script will still generate the files for the current check.
```
$ python3 route_approval_check_v2.py transit-90
------------------------------------------------------------------------------------------------------------------------
The Transit Gateway 'transit-90' is present / exists.
------------------------------------------------------------------------------------------------------------------------
File transit-90_connection_t90-t80_approved_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_t90-t80_total_approved_cidr_date_2024-05-28.csv has been created
File transit-90_connection_t90-t80_pending_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_t90-t80_total_pending_cidr_date_2024-05-28.csv has been created
File transit-90_connection_fake_approved_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_fake_total_approved_cidr_date_2024-05-28.csv has been created
File transit-90_connection_fake_pending_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_fake_total_pending_cidr_date_2024-05-28.csv has been created
File transit-90_connection_fake2_approved_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_fake2_total_approved_cidr_date_2024-05-28.csv has been created
File transit-90_connection_fake2_pending_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_fake2_total_pending_cidr_date_2024-05-28.csv has been created
File transit-90_connection_tr90-tr70_approved_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_tr90-tr70_total_approved_cidr_date_2024-05-28.csv has been created
File transit-90_connection_tr90-tr70_pending_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_tr90-tr70_total_pending_cidr_date_2024-05-28.csv has been created
------------------------------------------------------------------------------------------------------------------------
Golden List file golden_list/transit-90_connection_t90-t80_approved_cidr_list_golden_list.csv for approved CIDRs does not exist.
File golden_list/transit-90_connection_t90-t80_approved_cidr_list_golden_list.csv not found
Golden file golden_list/transit-90_connection_t90-t80_approved_cidr_list_golden_list.csv does not exist
File golden_list/transit-90_connection_fake_approved_cidr_list_golden_list.csv not found
Golden file golden_list/transit-90_connection_fake_approved_cidr_list_golden_list.csv does not exist
File golden_list/transit-90_connection_fake2_approved_cidr_list_golden_list.csv not found
Golden file golden_list/transit-90_connection_fake2_approved_cidr_list_golden_list.csv does not exist
File golden_list/transit-90_connection_tr90-tr70_approved_cidr_list_golden_list.csv not found
Golden file golden_list/transit-90_connection_tr90-tr70_approved_cidr_list_golden_list.csv does not exist
```
### Executing the script with "golden list" files available
Example of the output of executing the script for the existing Transit Gateway (in case the "golden list" files exist):
```
$ python3 route_approval_check_v2.py transit-90
------------------------------------------------------------------------------------------------------------------------
The Transit Gateway 'transit-90' is present / exists.
------------------------------------------------------------------------------------------------------------------------
File transit-90_connection_t90-t80_approved_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_t90-t80_total_approved_cidr_date_2024-05-28.csv has been created
File transit-90_connection_t90-t80_pending_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_t90-t80_total_pending_cidr_date_2024-05-28.csv has been created
File transit-90_connection_fake_approved_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_fake_total_approved_cidr_date_2024-05-28.csv has been created
File transit-90_connection_fake_pending_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_fake_total_pending_cidr_date_2024-05-28.csv has been created
File transit-90_connection_fake2_approved_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_fake2_total_approved_cidr_date_2024-05-28.csv has been created
File transit-90_connection_fake2_pending_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_fake2_total_pending_cidr_date_2024-05-28.csv has been created
File transit-90_connection_tr90-tr70_approved_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_tr90-tr70_total_approved_cidr_date_2024-05-28.csv has been created
File transit-90_connection_tr90-tr70_pending_cidr_list_date_2024-05-28.csv has been created
File transit-90_connection_tr90-tr70_total_pending_cidr_date_2024-05-28.csv has been created
------------------------------------------------------------------------------------------------------------------------
Connection name:  t90-t80
The number of approved CIDRs in the currrent-check of Route Approval feature: 2112
The number of approved CIDRs in the Golden List file: 4
Total approved CIDR number is greater than Golden-List count for connection:  t90-t80
------------------------------------------------------------------------------------------------------------------------
Connection name:  fake
The number of approved CIDRs in the currrent-check of Route Approval feature: 0
The number of approved CIDRs in the Golden List file: 1
Total approved CIDR number is lower than Golden-List count for connection:  fake
------------------------------------------------------------------------------------------------------------------------
Connection name:  fake2
The number of approved CIDRs in the currrent-check of Route Approval feature: 0
The number of approved CIDRs in the Golden List file: 1
Total approved CIDR number is lower than Golden-List count for connection:  fake2
------------------------------------------------------------------------------------------------------------------------
Connection name:  tr90-tr70
The number of approved CIDRs in the currrent-check of Route Approval feature: 0
The number of approved CIDRs in the Golden List file: 3
Total approved CIDR number is lower than Golden-List count for connection:  tr90-tr70
------------------------------------------------------------------------------------------------------------------------
CIDR DETAILS - Gateway: transit-90 Connection name: t90-t80
The CIDRs approved but not present in Golden List file golden_list/transit-90_connection_t90-t80_approved_cidr_list_golden_list.csv:
13.13.13.1/32
13.13.13.10/32
13.13.13.100/32
13.13.13.101/32
13.13.13.104/32
<..... output omitted .....>
7.7.7.40/32
7.7.7.41/32
7.7.7.5/32
7.7.7.6/32
7.7.7.7/32
7.7.7.8/32
7.7.7.9/32
Mismatch notification email sent successfully to <email@email>!
------------------------------------------------------------------------------------------------------------------------
CIDR DETAILS - Gateway: transit-90 Connection name: fake
The CIDRs present in Golden List golden_list/transit-90_connection_fake_approved_cidr_list_golden_list.csv file but not Approved by Route Approval feature:
8.0.0.0/24
Mismatch notification email sent successfully to <email@email>!
------------------------------------------------------------------------------------------------------------------------
CIDR DETAILS - Gateway: transit-90 Connection name: fake2
The CIDRs present in Golden List golden_list/transit-90_connection_fake2_approved_cidr_list_golden_list.csv file but not Approved by Route Approval feature:
8.0.0.0/24
Mismatch notification email sent successfully to <email@email>!
------------------------------------------------------------------------------------------------------------------------
CIDR DETAILS - Gateway: transit-90 Connection name: tr90-tr70
The CIDRs present in Golden List golden_list/transit-90_connection_tr90-tr70_approved_cidr_list_golden_list.csv file but not Approved by Route Approval feature:
7.0.0.0/16
7.1.0.0/16
7.7.7.7/32
Mismatch notification email sent successfully to <email@email>!
```
### Executing the script for non-existent Transit Gateway
Example of the output of executing the script for the non-existent Transit Gateway:
```
$ python3 route_approval_check2.py  transit-1234
------------------------------------------------------------------------------------------------------------------------
The Transit Gateway 'transit-1234' is not present / does not exist.
------------------------------------------------------------------------------------------------------------------------
```
### E-mail notifications
In case the list of approved CIDRs between current check and "golden list" is not equal -> script sends a notification e-mail(s).
Example of the e-mail notification for approved CIDRs (e-mail title: Aviatrix - Route Approval - Approved CIDRs Mismatch detected):
```
Total number of approved CIDRs changed for gateway: transit-90 and connection: tr90-tr70.



The following CIDRs are approved by Route Approval feature but not present in Golden List:
7.0.0.0/16
7.1.0.0/16
7.7.7.7/32

```
