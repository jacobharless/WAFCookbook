# WAFCookbook

##  Prereqs
Have an OCI account (Cloud.oracle.com). 

Have access to Edge Services in OCI. 

Provision an instance in OCI region. 

Download OCI Client
`bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"` 

## Step 1- Launch Webapp and add it to a domain

**Install docker on this VM.**

`yum install docker-engine`

`systemctl start docker`

`systemctl enable docker`

Pull docker image of open source software ORAWASP Juice Shop application. Run this 
application on host port 80

`Run docker pull bkimminich/juice-shop`
`Run docker run --rm -p 80:3000 bkimminich/juice-shop`

Ensure that security list used by the instance has appropriate rule so that port 80 is 
accessible from internet. 

Register to website www.freenom.com to create domain (For ex.) `www.trywaf.tk`
  
## Step 2 Tie together OCI WAF with Web application

Ensure that DNS A record is created for your domain that resolve to public IP of OCI instance.

Create WAF policy in OCI Edge services to protect the origin of this application i.e. the 
public IP of instance with default port 80. 

Note down the CNAME target of this WAF policy. 

Now make CNAME record in the DNS of your domain that maps to `www.trywaf.tk`

If you now access `www.trywaf.tk` because of above CNAME entry, traffic reaches to 
WAF node, WAF policy is applied and then it reaches to your origin i.e. OCI instance.

Access OWASP juice shop application. This application purposefully has many security 
vulnerabilities. Attack certain selected security vulnerabilities and observe that 
application is successfully attacked.

Enable WAF protection rules to this WAF policy.

Access OWASP juice shop application using WAF protected URL. Observe that 
application is now protected from that attack. This proves WAF policy is working to 
prevent such attack. When using URL with public IP application works without having 
protection from vulnerabilities
  
## Step 3 - Setting WAF Policies

### Create WAD Policy:
Select the region and compartment where the policy should be maintained (there is no constraint around the WAF co-existing with Load Balancing or other application resources in Oracle Cloud Infrastructure.

Open the navigation menu. Under Solutions, Platform and Edge, go to Edge Services and click WAF Policies.

Click Create WAF Policy.
In the Create WAF Policy dialog box, enter the following:
Policy Name: A unique name for the policy. Avoid entering confidential information.
Domains:
Primary Domain: The fully qualified domain name (FQDN) of the application where the policy will be applied.
Additional Domains: (Optional) Subdomains where the policy will be applied.
WAF Origin: The host or IP address of the public internet facing application that is being protected by the application. Origin Name: A unique name for the origin.
URI - Enter the public facing endpoint (IPv4 or FQDN) of the application.
HTTPS Port: The port used for secure HTTP connection. The default port is 443.
HTTP Port: The HTTP port the origin listens on. The default port is 80. 
Header(s): (Optional)
Header Name: The name displayed in the HTTP request header and the header value that can be added and passed to  the origin server with all requests.
Header Value: Specifies the data requested by the header.

Click Create WAF Policy. The WAF Policy overview appears. Expect the policy to become active within 15 minutes of creation.

### Add Access Control rules to Policy

#### GEOIP Rule
Name rule GEOIP
Select under Rule Condition- `Country/Region is not` Country Region -`United States`
Select Action- BLOCK
Use Prefilled options for Block actions

#### Header Access Rule
Name rule HeadeAccessRule
Rule Condition-> `HTTP Header contains`
Header-> `name:blockme`
Select Action: Block
Block Action: Show Error Page

#### URL ends in Rule
Name Rule:URLAccessRule
Rule Condition-> `URL Part Ends with`
Header-> `blockthis`
Additional Condition:
Rule Condition-> `URL Part Starts with`
Header-> `foo`
Select Action: Block
Block Action: Show Error Page

#### Publish
Press Publish All

### Add Protection Rules to Policy
Go to Protection Rules under your policy.
Filter by Rule ID (type these numbers in)
973335
933161
960335
981242
90004
90001
941140
941110
911100
950007
Select All (Check Box next to ID)
Select Actions: Block

### Bot Management
 #### Javascript Challenge
  Add Javascript Challenge
  Enable Javascript Challenge
  Set Action to Block
  Add
  
  #### Captcha Challenge
  Add Captcha Challenge
  URL-> /captcha
  Add
  
  Publish

###
Set up Client with OCI
`oci waas threat-feed update --waas-policy-id OCID --threat-feeds '[{"key":"<key>,"action","BLOCK"}]'`

#### User Agent
### Trigger rules


#### Header Rule:
Download ModHeader
Enter header name: name and value:blockme (Header Rule)
Try to access
#### URL ends with/start with
Append to your web app URL
/foo?name=blockthis (URL starts/ends with)

#### GEOIP
Use tor browser to access website (GeoIP) from country outside US

#### SQL Injection
5;return(true);

#### PHP Vulnerability
process.exit()




##  Troubleshooting
  
502 Error -  Is having trouble with WAF connecting to Origin
