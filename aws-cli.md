### Commands, scripts and other magics


### Show all VPC Names
aws ec2 describe-vpcs|jq -r '.Vpcs[].Tags[] | select(.Key=="Name") | .Value'


### Show public dns from bastion host
aws ec2 describe-instances|jq -r '.Reservations[].Instances[] | select(.KeyName == "bastion") | .PublicDnsName'


### Execute remote ping to google in remote host
ssh -i "CLIENT-X.pem" ubuntu@$(aws ec2 describe-instances|jq -r '.Reservations[].Instances[]| select(.Tags[].Value == "emcali-connector") | .PublicDnsName') 'ping 8.8.8.8 -c5


### Describe all instances on profile
aws ec2 describe-instances
 # to avoid use --profile you can export current profile
 # export AWS_PROFILE=YOUR-PROFILE


### List instances by Name and ID
aws ec2 describe-instances  --output text --query 'Reservations[].Instances[].[InstanceId, Tags[?Key==`Name`].Value|[0]]'


### Filter instances by ID,Name and tag ShutingOff=True
aws ec2 describe-instances  --output text  --filters "Name=tag:ShutingOff,Values=True"  --query 'Reservations[].Instances[].[InstanceId, Tags[?Key==`Name`].Value|[0]]'


### Mismo que el anterior pero mostrando el estado running
aws ec2 describe-instances  --output text  --filters "Name=tag:ShutingOff,Values=True" "Name=instance-state-code,Values=80"  --query 'Reservations[].Instances[].[State.Name,InstanceId, Tags[?Key==`Name`].Value|[0]]'


### Count instance type group by type
aws ec2 describe-instances --region us-east-1 --filters "Name=instance-type,Values=*" --query "Reservations[].Instances[].{InstanceType:InstanceType}" | jq "group_by(.InstanceType) | map({(.[0].InstanceType):length})"


### List SG with 0.0.0.0 rule
aws ec2 describe-security-groups --filters "Name=ip-permission.to-port,Values=22"  --query 'SecurityGroups[?length(IpPermissions[?ToPort==`22` && contains(IpRanges[].CidrIp
, `0.0.0.0/0`)]) > `0`].{GroupName: GroupName, TagName: Tags[?Key==`Name`].Value | [0]}' --output table


### Filter all profiles
grep '^[[]profile' <~/.aws/config | awk '{print $2}' | sed 's/]$//'



### List all SG with SSH port on all profiles.
for profile in $(grep '^[[]profile' <~/.aws/config | awk '{print $2}' | sed 's/]$//'); do  aws --profile $profile ec2 describe-security-groups --filters Name=ip-permission.from-port,Values=22 Name=ip-permission.to-port,Values=22  --query 'SecurityGroups[*].{Name:GroupName}' --output table| sed 's/DescribeSecurityGroups/Profile: '$profile'/g'  ; done



### List all RDS on each profile
for profile in $(grep '^[[]profile' <~/.aws/config | awk '{print $2}' | sed 's/]$//'); do echo -e "\nProfile: "$profile ; aws --profile $profile rds describe-db-instances --query 'DBInstances[*].[DBInstanceArn,Engine,DBInstanceIdentifier]' --output table   ; done


### List all SG with 0.0.0.0
for profile in $(grep '^[[]profile' <~/.aws/config | awk '{print $2}' | sed 's/]$//'); do echo -e "\nProfile: "$profile ; aws --profile $profile ec2  describe-security-groups --filter Name=ip-permission.protocol,Values=-1 Name=ip-permission.cidr,Values='0.0.0.0/0' --query "SecurityGroups[*].{Name:GroupName,ID:GroupId}" --output table  ;  done


### Filter all instances by tag where tag_name=Project and tag_key=CLIENT-X and finally filter by ID
aws ec2 describe-instances --filters "Name=tag:Project,Values=CLIENT-X" |jq -r .Reservations[].Instances[].InstanceId


### Filter instances by tag name and tag ShutingOff
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{Instance:InstanceId}' --filters "Name=tag:Name,Values=CLIENT-X*" "Name=tag:ShutingOff,Values=False" --output text
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId, Tags[?Key==`Name`].Value]' --output table


### Tag instances
aws ec2 create-tags --resources i-00000000000 --tags Key=hello,Value=world Key=hi,Value=moon


### Mostrar el estado de la vpn filtrando por el nombre de la VPN llamada CLIENT-X
aws ec2 describe-vpn-connections  --filters "Name=tag:Name,Values=CLIENT-X" |jq -r '.VpnConnections[].State'


### Mostrar las rutas estaticas de la VPN con nombre CLIENT-X VPN
aws ec2 describe-vpn-connections  --filters "Name=tag:Name,Values=CLIENT-X" |jq -r '.VpnConnections[].Routes[].DestinationCidrBlock'


### Mostrar los tag-nombres de las VPNs
aws ec2 describe-vpn-connections --query "VpnConnections[*].Tags[?Key=='Name'].Value[]"|jq -r '.[]'
aws ec2 describe-vpn-connections --query 'VpnConnections[*].Tags[?Key==`Name`].Value[]'|jq -r '.[]'


### Neo4j (pending...)
aws ec2 describe-vpcs|jq -r '.Vpcs[].Tags[] | select(.Key=="Name") | .Value' |awk {'print "CREATE (",$1":VPC { name: +"$1"+ })" '}|sed "s/-/_/g; s/+/'/g"
