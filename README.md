The provided PowerShell script is designed to retrieve firewall rules from a specified Azure Firewall Policy and its associated Rule Collection Groups, then export those rules to a CSV file. Here are a few comments and explanations to make the code clearer:

```powershell
# Provide Input: Firewall Policy Name, Resource Group, and Rule Collection Group Names
$fpname = "firewallpolicyname"
$fprg = "resouece-group-name"
$rcgNames = @("DefaultNetworkRuleCollectionGroup", "o365_rulecollectiongroup", "DefaultApplicationRuleCollectionGroup") # Add your rule collection group names here

# Get the Firewall Policy
$fp = Get-AzFirewallPolicy -Name $fpname -ResourceGroupName $fprg

# Initialize an array to store the rules
$returnObj = @()

# Loop through each specified Rule Collection Group
foreach ($rcgName in $rcgNames) {
    # Retrieve the Rule Collection Group
    $rcg = Get-AzFirewallPolicyRuleCollectionGroup -Name $rcgName -AzureFirewallPolicy $fp

    # Loop through each Rule Collection within the Rule Collection Group
    foreach ($rulecol in $rcg.Properties.RuleCollection) {
        # Loop through each rule in the Rule Collection
        foreach ($rule in $rulecol.rules) {
            # Prepare a hashtable with the rule details
            $properties = [ordered]@{
                RuleCollectionGroupName = $rcg.Name
                RuleCollectionName = $rulecol.Name
                RulePriority = $rulecol.Priority
                ActionType = $rulecol.Action.Type
                RuleCollectionType = $rulecol.RuleCollectionType
                Name = $rule.Name
                Protocols = $rule.protocols -join ", "
                SourceAddresses = $rule.SourceAddresses -join ", "
                DestinationAddresses = $rule.DestinationAddresses -join ", "
                SourceIPGroups = $rule.SourceIPGroups -join ", "
                DestinationIPGroups = $rule.DestinationIPGroups -join ", "
                DestinationPorts = $rule.DestinationPorts -join ", "
                DestinationFQDNs = $rule.DestinationFQDNs -join ", "
            }
            # Create a new object with the rule details and add it to the array
            $obj = New-Object psobject -Property $properties
            $returnObj += $obj
        }
    }
}

# Export the rules to a CSV file
$returnObj | Export-Csv "firewall-collections.csv" -NoTypeInformation
```

### Explanation:
1. **Input Variables:** The script starts by defining the input variables - the name of the Firewall Policy, the Resource Group it belongs to, and an array of Rule Collection Group names.
2. **Retrieve Firewall Policy:** The `Get-AzFirewallPolicy` cmdlet is used to get the specified Firewall Policy.
3. **Initialize Array:** An empty array `$returnObj` is initialized to store the rule details.
4. **Loop Through Rule Collection Groups:** The script loops through each Rule Collection Group name provided in the `$rcgNames` array.
5. **Retrieve Rule Collection Group:** For each Rule Collection Group, the `Get-AzFirewallPolicyRuleCollectionGroup` cmdlet retrieves its details.
6. **Loop Through Rule Collections and Rules:** The script then loops through each Rule Collection within the Rule Collection Group, and each rule within the Rule Collection.
7. **Prepare Rule Details:** For each rule, a hashtable is created with the details of the rule such as its name, protocols, source addresses, etc.
8. **Create Object and Add to Array:** A new PowerShell object is created from the hashtable and added to the `$returnObj` array.
9. **Export to CSV:** Finally, the array of rule details is exported to a CSV file named "firewall-collections.csv".

This script should work as expected, but make sure to have the necessary Azure PowerShell module installed and authenticated to your Azure account.
