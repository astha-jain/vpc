---

copyright:
  years: 2022
lastupdated: "2022-09-21"

keywords: auto-negotiation, ciphers, upgrading ciphers, migrating ciphers

subcollection: vpc

---

{{site.data.keyword.attribute-definition-list}}

# Upgrading weak cipher suites on a VPN gateway
{: #upgrading-weak-ciphers}

To maintain security best practices and minimize security vulnerabilities, VPN for VPC now supports an enhanced cipher suite, providing new algorithms and removing weak algorithms to meet customer compliance requirements. VPN gateways that use weak ciphers put networks and data at risk and compromise compliance, requiring an upgrade to the secure cipher suite.
{: shortdesc}

As of 20 September 2022, the following VPN IKE and IPsec ciphers are now deprecated:

- Authentication algorithms `md5` and `sha1`
- Encryption algorithm `triple_des`
- Diffie–Hellman groups `2` and `5`

You have until 13 December 2022 to transition to more secure ciphers. After this date:

* VPN connections that use deprecated ciphers stop working. 
* VPN connections that use an auto-negotiation policy are forced to upgrade to the [enhanced auto-negotiation policy](/docs/vpc?topic=vpc-using-vpn#policy-negotiation). 
* VPN connections that use custom IKE or IPsec policies that contain deprecated ciphers are disabled.

To prevent affected connections from becoming disconnected, take steps to upgrade your VPN connection to secure ciphers now. Also, if your disaster recovery plan or API/CLI-based automation references these deprecated ciphers, take steps to upgrade them.
{: important}

## Upgrading the auto-negotiation policy
{: #upgrade-vpn-with-old-auto}

Complete the following procedure to upgrade your VPN to the enhanced auto-negotiation policy. You can upgrade the auto-negotiation policy using the UI, CLI, or API.

### Before you begin
{: #upgrade-vpn-old-auto-before-begin}

You will experience a network outage during the upgrade. The duration of the outage depends on the interval between the disable and re-enable actions to re-establish the VPN connection. It is recommended that you plan a maintenance window for this upgrade. 
{: important}

Before you upgrade, review the following information:

* By default, the new auto-negotiation policy is used for newly created VPN connections. For more information, see [About policy negotiation](/docs/vpc?topic=vpc-using-vpn#policy-negotiation).
* Because IBM Cloud auto-negotiation uses IKEv2, the on-prem device must also use IKEv2. If your on-prem device does not support IKEv2, see [upgrading VPN from a custom IKE or IPsec policy](/docs/vpc?topic=vpc-upgrading-weak-ciphers#upgrade-vpn-with-custom-policy). 
* It is a good idea to first configure your on-prem VPN gateway peer to replace the weak ciphers for Phase 1 and Phase 2 negotiation with the secure ciphers that are described in [policy negotiation](/docs/vpc?topic=vpc-using-vpn#policy-negotiation). Then, upgrade the VPN gateway to use the enhanced auto-negotiation policy. This step might also reduce the outage time. 

For an existing VPN connection using the old auto-negotiation policy (created before 20 September 2022), complete the following steps to upgrade to the new auto-negotiation policy.

### Upgrading the auto-negotiation policy by using the UI
{: #upgrade-vpn-old-auto-procedure-ui}
{: ui}

To upgrade the auto-negotiation policy by using the UI, follow these steps:

1. From the VPNs for VPC page, select **Site-to-site gateways > VPN gateways**.
1. Select the VPN gateway that contains the VPN connection that you want to upgrade.
1. Highlight the row of the VPN connection in the VPN connection table, then check that the **State** switch is enabled by default.
1. Toggle the **State** switch to disable the connection. If your **State** switch is already disabled, skip this step.
1. Toggle the **State** switch to re-enable the connection.

### Upgrading the auto-negotiation policy by using the CLI
{: #upgrade-vpn-old-auto-procedure-ui-cli}
{: cli}

Before you begin, make sure to [set up your CLI environment](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference).

To upgrade the auto-negotiation policy by using the CLI, follow these steps:

1. Store the VPN gateway ID (or name) and VPN connection ID (or name) variables to be used in the CLI command, for example:

   `vpn_gateway` - Find the VPN gateway ID (or name) by using the [get VPN gateways](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference&interface=cli#vpn-gateways) command, and then populate the variable.
   
   `connection` - Find the VPN gateway connection ID (or name) by using the [get VPN gateway connections](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference&interface=cli#vpn-gateway-connections) command, and then populate the variable.

    ```sh
    export vpn_gateway=<vpn_gateway_id_or_name>
    export connection=<vpn_gateway_connection_id_or_name>
    ```
    {: codeblock}

1. Set **--admin-state-up value** to `false` to disable the VPN connection, then wait for `Status` to change to `down`. If your **--admin-state-up value** is already set to `false`, skip this step.

    ```sh
    # set the admin-state-up value to false to disable the VPN connection
    ibmcloud is vpn-gateway-connection-update $vpn_gateway $connection --admin-state-up false

    # check the VPN connection status to be changed to down
    ibmcloud is vpn-gateway-connection $vpn_gateway $connection
    ```
    {: codeblock}

1. Set **--admin-state-up value** to `true` to re-enable the VPN connection, then wait for the `Status` to change to `up`.

    ```sh
    # set the admin-state-up value to true to re-enable the VPN connection
    ibmcloud is vpn-gateway-connection-update $vpn_gateway $connection --admin-state-up true

    # check the VPN connection status to be changed to up
    ibmcloud is vpn-gateway-connection $vpn_gateway $connection
    ```
    {: codeblock}

### Upgrading the auto-negotiation policy by using the API
{: #upgrade-vpn-old-auto-procedure-ui-api}
{: api}

To upgrade the auto-negotiation policy by using the API, follow these steps:

1. Set up your [API environment](/docs/vpc?topic=vpc-set-up-environment#api-prerequisites-setup) with the right variables.

1. Store the VPN gateway ID and VPN connection ID variables to be used in the API, for example:

   `vpn_gateway_id` - Find the VPN gateway ID by using the [get VPN gateways](/apidocs/vpc/latest#list-vpn-gateways) API, and then populate the variable.
   
   `vpn_connection_id` - Find the VPN gateway connection ID by using the [get VPN gateway connections](/apidocs/vpc/latest#list-vpn-gateway-connections) API, and then populate the variable.

    ```sh
    export vpn_gateway_id=<your_vpn_gateway_id>
    export vpn_connection_id=<your_vpn_gateway_connection_id>
    ```
    {: codeblock}

1. When all variables are initiated, set the **admin_state_up** parameter to `false` to disable the VPN connection, then wait for the `Status` to change to `down`. If your **admin_state_up** is already set to `false`, skip this step.

   ```sh
   # set the admin_state_up parameter to false to disable the VPN connection
   curl -X PATCH "$vpc_api_endpoint/v1/vpn_gateways/$vpn_gateway_id/connections/$vpn_connection_id?version=$api_version&generation=2" \
      -H "Authorization: $iam_token" \
      -d '{
         "admin_state_up": false
      }'

   # check the VPN connection status to be changed to down
   curl -X GET "$vpc_api_endpoint/v1/vpn_gateways/$vpn_gateway_id/connections/$vpn_connection_id?version=$api_version&generation=2" \
      -H "Authorization: $iam_token"
   ```
   {: codeblock}

1. Set the **admin_state_up** parameter to `true` to re-enable the VPN connection, then wait for the `Status` to change to `up`.

   ```sh
   # set the admin_state_up parameter to true to re-enable the VPN connection
   curl -X PATCH "$vpc_api_endpoint/v1/vpn_gateways/$vpn_gateway_id/connections/$vpn_connection_id?version=$api_version&generation=2" \
      -H "Authorization: $iam_token" \
      -d '{
         "admin_state_up": true
      }'

   # check the VPN connection status to be changed to up
   curl -X GET "$vpc_api_endpoint/v1/vpn_gateways/$vpn_gateway_id/connections/$vpn_connection_id?version=$api_version&generation=2" \
      -H "Authorization: $iam_token"
   ```
   {: codeblock}

## Upgrading a VPN from a custom IKE or IPsec policy
{: #upgrade-vpn-with-custom-policy}

Complete the following procedure to upgrade a VPN from a custom IKE or IPsec policy. 

### Before you begin
{: #upgrade-vpn-custom-policy-before-begin}

On 20 September 2022, VPN for VPC IKE and IPsec weak ciphers were deprecated. To upgrade a VPN connection that was created using a custom IKE or IPsec policy that contains weak ciphers, complete the following steps.

You will experience a network outage during the upgrade. The duration of the outage depends on the time that it takes to update the weak ciphers and to re-establish the VPN connection. It is recommended that you plan a maintenance window for this upgrade.
{: important}

Before you begin, it is a good idea to first configure your on-prem VPN gateway peer to contain both the weak and secure ciphers for Phase 1 and Phase 2 negotiation. Then, change the IBM VPN gateway to remove the use of the weak ciphers by following these steps. Afterward, remove the weak ciphers from the on-prem VPN gateway. This step might also reduce the outage time.

### Upgrading a VPN from a custom IKE policy by using the UI
{: #upgrade-vpn-custom-ike-policy-procedure-ui}
{: ui}

To upgrade the IKE policy by using the UI, follow these steps:

1. From the VPNs for VPC page, select **Site-to-site gateways > IKE policies**.
1. Select the IKE policy configured in the VPN connection that you want to upgrade.
1. Highlight the row of the IKE policy in the table, then click **Edit** from the Actions menu ![Actions menu](images/overflow.png).
1. Update the following algorithms to replace the weak ciphers with secure ones:

   * **Encryption** - Encryption algorithm to use for IKE Phase 1. One of: `aes128`, `aes192`, `aes256`.
   * **Authentication** - Authentication algorithm to use for IKE Phase 1. One of: `sha256`, `sha384`, `sha512`.
   * **Diffie-Hellman group** - DH group to use for IKE Phase 1. One of: `14`, `15`, `16`, `17`, `18`, `19`, `20`, `21`, `22`, `23`, `24`, `31`.
1. Click **Save**.

### Upgrading a VPN from a custom IPsec policy by using the UI
{: #upgrade-vpn-custom-ipsec-policy-procedure-ui}
{: ui}

To upgrade a custom IPsec policy by using the UI, follow these steps:

1. From the VPNs for VPC page, select **Site-to-site gateways > IPsec policies**.
1. Select the IPsec policy configured in the VPN connection that you want to upgrade.
1. Highlight the row of the IPsec policy in the table, then click **Edit** from the Actions menu ![Actions menu](images/overflow.png).
1. Update the following algorithms to replace the weak ciphers with secure ones:

   * **Encryption** - Encryption algorithm to use for IKE Phase 2. One of: `aes128`, `aes192`, `aes256`, `aes128gcm16`, `aes192gcm16`, `aes256gcm16`.
   * **Authentication** - Authentication algorithm to use for IKE Phase 2. One of: `sha256`, `sha384`, `sha512`, `disabled`.

      The authentication is `disabled` when combined-mode encryption `aes128gcm16`, `aes192gcm16`, or `aes256gcm16` is selected.
      {: note}
      
   * **Diffie-Hellman Group (if PFS is enabled)** - DH group to use for IKE Phase 2 key exchange. One of: `group_14`, `group_15`, `group_16`, `group_17`, `group_18`, `group_19`, `group_20`, `group_21`, `group_22`, `group_23`, `group_24`, `group_31`.

      The Diffie-Hellman Group is `disabled` when PFS is disabled.
      {: note}
      
1. Click **Save**.

### Upgrading a VPN from a custom IKE policy by using the CLI
{: #upgrade-vpn-custom-ike-policy-procedure-cli}
{: cli}

Before you begin, make sure to [set up your CLI environment](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference).

To upgrade a custom IKE policy by using the CLI, following these steps:

1. Find and store the IKE policy ID or name variable to be used in your CLI code, for example:

   `ike_policy` - Find the ID (or name) of IKE policy that contains weak ciphers by using the [list IKE policies](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference&interface=cli#ike-policies) command, and then populate the variable.

    ```sh
    export ike_policy=<your_ike_policy_id_or_name>
    ```
    {: codeblock}

1. Find and replace IKE policies **authentication_algorithm**, **dh_group**, and **encryption_algorithm** to use secure ciphers, and populate these variables in your CLI code.

   `authentication_algorithm` - The authentication algorithm. One of: `sha256`, `sha384`, `sha512`. 
   
   `dh_group` - The Diffie-Hellman group. One of: `14`, `15`, `16`, `17`, `18`, `19`, `20`, `21`, `22`, `23`, `24`, `31`.
   
   `encryption_algorithm` - The encryption algorithm. One of: `aes128`, `aes192`, `aes256`.

    ```sh
    export authentication_algorithm=<your_secure_authentication_algorithm>
    export dh_group=<your_secure_dh_group>
    export encryption_algorithm=<your_secure_encryption_algorithm>
    ```
    {: codeblock}

1. Replace IKE policies **--authentication-algorithm value**, **--dh-group value**, and **--encryption-algorithm value** with secure ciphers in your CLI code.

   ```sh
   ibmcloud is ike-policy-update $ike_policy
      [--authentication-algorithm $authentication_algorithm]
      [--dh-group $dh_group]
      [--encryption-algorithm $encryption_algorithm]
   ```
   {: pre}

   Command examples:

   - Initiate the IKE policy variables:
      - `export ike_policy=my-ike-policy`
      - `export authentication_algorithm=sha256`
      - `export dh_group=14`
      - `export encryption_algorithm=aes128`
   - Update an IKE policy by using SHA 256 authentication:
      - `ibmcloud is ike-policy-update $ike_policy --authentication-algorithm $authentication_algorithm`
   - Update an IKE policy by using DH Group 14:
      - `ibmcloud is ike-policy-update $ike_policy --dh-group $dh_group`
   - Update an IKE policy by using AES 128 encryption:
      - `ibmcloud is ike-policy-update $ike_policy --encryption-algorithm $encryption_algorithm`
   - Update an IKE policy by using SHA 256 authentication, DH Group 14, AES 128 encryption:
      - `ibmcloud is ike-policy-update $ike_policy --authentication-algorithm $authentication_algorithm --dh-group $dh_group --encryption-algorithm $encryption_algorithm`

### Upgrading a VPN from a custom IPsec policy by using the CLI
{: #upgrade-vpn-custom-ipsec-policy-procedure-cli}
{: cli}

Before you begin, make sure to [set up your CLI environment](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference).

To upgrade a custom IPsec policy by using the CLI, follow these steps:

1. Find and store the IPsec policy ID (or name) variable to be used in your CLI code, for example:

   `ipsec_policy` - Find the ID (or name) of IPsec policy that contains weak ciphers by using the [list IPsec policies](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference&interface=cli#ipsec-policies) command, and then populate the variable:

    ```sh
    export ipesc_policy=<your_ipsec_policy_id_or_name>
    ```
    {: codeblock}

1. Find and replace IPsec policies **authentication_algorithm**, **dh_group**, and **encryption_algorithm** to use secure ciphers, and populate these variables in your CLI code.

   `authentication_algorithm` - The authentication algorithm. One of: `sha256`, `sha384`, `sha512`.
   
   `pfs` - Perfect Forward Secrecy. One of: `disabled`, `group_14`, `group_15`, `group_16`, `group_17`, `group_18`, `group_19`, `group_20`, `group_21`, `group_22`, `group_23`, `group_24`, `group_31`.
   
   `encryption_algorithm` - The encryption algorithm. One of: `aes128`, `aes192`, `aes256`.

      The `authentication_algorithm` must be `disabled` if and only if `encryption_algorithm` is `aes128gcm16`, `aes192gcm16`, or `aes256gcm16`.
      {: note}

    ```sh
    export authentication_algorithm=<your_secure_authentication_algorithm>
    export pfs=<your_secure_pfs_group>
    export encryption_algorithm=<your_secure_encryption_algorithm>
    ```
    {: codeblock}

1. Replace IPsec policies **--authentication-algorithm value**, **--pfs value**, and **--encryption-algorithm value** with secure ciphers in your CLI code.

   ```sh
   ibmcloud is ipsec-policy-update $ipesc_policy
      [--authentication-algorithm $authentication_algorithm]
      [--pfs $pfs]
      [--encryption-algorithm $encryption_algorithm]
   ```
   {: pre}

   Command examples:

   - Initiate the IPsec policy variables:
      - `export ipesc_policy=my-ipsec-policy`
      - `export authentication_algorithm=sha256`
      - `export pfs=14`
      - `export encryption_algorithm=aes128`
   - Update an IPsec policy by using SHA 256 authentication:
      `ibmcloud is ipsec-policy-update $ipesc_policy --authentication-algorithm $authentication_algorithm`
   - Update an IPsec policy by using PFS with DH Group 14:
      `ibmcloud is ipsec-policy-update $ipesc_policy --pfs $pfs`
   - Update an IPsec policy by using AES 128 encryption:
      `ibmcloud is ipsec-policy-update $ipesc_policy --encryption-algorithm $encryption_algorithm`
   - Update an IPsec policy by using SHA 256 authentication, AES 128 encryption, and PFS with DH Group 14:
      `ibmcloud is ipsec-policy-update $ipesc_policy --authentication-algorithm $authentication_algorithm --pfs $pfs --encryption-algorithm $encryption_algorithm`

### Upgrading a VPN from a custom IKE policy by using the API
{: #upgrade-vpn-custom-ike-policy-procedure-api}
{: api}

Before you begin, make sure to [set up your API environment](/docs/vpc?topic=vpc-set-up-environment#api-prerequisites-setup).

To upgrade a custom IKE policy by using the API, follow these steps:

1. Find and store the IKE policy ID variable to be used in your API code, for example:

   `ike_policy_id` - Find the ID of IKE policy that contains weak ciphers by using the [list IKE policies](/apidocs/vpc/latest#list-ike-policies) command, and then populate the variable:

    ```sh
    curl -X PATCH "$vpc_api_endpoint/v1/ike_policies/$ike_policy_id?version=$api_version&generation=2" \
      -H "Authorization: $iam_token" \

    export ike_policy_id=<your_ike_policy_id>
    ```
    {: codeblock}

1. Find and replace IKE policies **authentication_algorithm**, **dh_group**, and **encryption_algorithm** to use secure ciphers, and populate these variables in your API code.
 
   `authentication_algorithm` - The authentication algorithm. One of: `sha256`, `sha384`, `sha512`.
   
   `dh_group` - The Diffie-Hellman group. One of: `14`, `15`, `16`, `17`, `18`, `19`, `20`, `21`, `22`, `23`, `24`, `31`.
   
   `encryption_algorithm` - The encryption algorithm. One of: `aes128`, `aes192`, `aes256`.

    ```sh
    export authentication_algorithm=<your_secure_authentication_algorithm>
    export dh_group=<your_secure_dh_group>
    export encryption_algorithm=<your_secure_encryption_algorithm>
    ```
    {: codeblock}

1. When IKE policy variables are initiated, replace the IKE policy with secure ciphers in your API code.

   ```sh
   curl -X PATCH "$vpc_api_endpoint/v1/ike_policies/$ike_policy_id?version=$api_version&generation=2" \
      -H "Authorization: $iam_token" \
      -d "{
         'authentication_algorithm': $authentication_algorithm,
         'dh_group': $dh_group,
         'encryption_algorithm': $encryption_algorithm
      }"
   ```
   {: codeblock}

### Upgrading a VPN from a custom IPsec policy by using the API
{: #upgrade-vpn-custom-ipsec-policy-procedure-api}
{: api}

Before you begin, make sure to [set up your API environment](/docs/vpc?topic=vpc-set-up-environment#api-prerequisites-setup).

To upgrade the IPsec policy by using the API, follow these steps:

1. Find and store IPsec policy ID variable to be used in your API code, for example:

   `ipsec_policy_id` - Find the IPsec policy ID by using the [list IPsec policies](/apidocs/vpc/latest#list-ipsec-policies) command and then populate the variable:

    ```sh
    export ipsec_policy_id=<your_ipsec_policy_id>
    ```
    {: codeblock}

1. Find and replace IPsec policies **authentication_algorithm**, **pfs**, and **encryption_algorithm** to use secure ciphers and populate these variables in your API code.

   `authentication_algorithm` - The authentication algorithm. One of: `sha256`, `sha384`, `sha512`.
   
   `pfs` - Perfect Forward Secrecy. One of: `disabled`, `group_14`, `group_15`, `group_16`, `group_17`, `group_18`, `group_19`, `group_20`, `group_21`, `group_22`, `group_23`, `group_24`, `group_31`.
   
   `encryption_algorithm` - The encryption algorithm. One of: `aes128`, `aes192`, `aes256`.

      The `authentication_algorithm` must be `disabled` if and only if `encryption_algorithm` is `aes128gcm16`, `aes192gcm16`, or `aes256gcm16`.
      {: note}

    ```sh
    export authentication_algorithm=<your_secure_authentication_algorithm>
    export pfs=<your_secure_pfs_group>
    export encryption_algorithm=<your_secure_encryption_algorithm>
    ```
    {: codeblock}

1. When the IPsec policy variables are initiated, replace the IPsec policy with secure ciphers in your API code.

   ```sh
   curl -X PATCH "$vpc_api_endpoint/v1/ipsec_policies/$ipsec_policy_id?version=$api_version&generation=2" \
      -H "Authorization: $iam_token" \
      -d "{
         'authentication_algorithm': $authentication_algorithm,
         'encryption_algorithm': $encryption_algorithm,
         'pfs': $pfs
      }"
   ```
   {: codeblock}