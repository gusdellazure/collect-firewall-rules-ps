# collect-firewall-rules-ps

CODE:

# Provide Input: Firewall Policy Name, Resource Group, and Rule Collection Group Names
$fpname = "fwpolicyprodavd1"
$fprg = "RG-VNET-AVD-PROD-VA-01"
$rcgNames = @("DefaultNetworkRuleCollectionGroup", "o365_rulecollectiongroup", "DefaultApplicationRuleCollectionGroup") # Add your rule collection group names here

# Get the Firewall Policy
$fp = Get-AzFirewallPolicy -Name $fpname -ResourceGroupName $fprg

# Initialize an array to store the rules
$returnObj = @()

# Loop through each specified Rule Collection Group
foreach ($rcgName in $rcgNames) {
    $rcg = Get-AzFirewallPolicyRuleCollectionGroup -Name $rcgName -AzureFirewallPolicy $fp
    foreach ($rulecol in $rcg.Properties.RuleCollection) {
        foreach ($rule in $rulecol.rules) {
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
            $obj = New-Object psobject -Property $properties
            $returnObj += $obj
        }
    }
}

# Export the rules to a CSV file
$returnObj | Export-Csv "firewall-collections.csv" -NoTypeInformation
