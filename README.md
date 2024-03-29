# WAFCookbook

##  Prereqs
Have an OCI account (Cloud.oracle.com). 

Have access to Edge Services in OCI. 

Provision an instance in OCI region. 

Download OCI Client
`bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"` 

## Step 1 - Launch Webapp and add it to a domain

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
  
## Step 2 - Tie together OCI WAF with Web application

Ensure that DNS A record is created for your domain that resolve to public IP of OCI instance.


### Create WAF Policy in OCI:
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

### Tie CNAME record to your domain
Note down the CNAME target of this WAF policy. 

Now make CNAME record in the DNS of your domain that maps to `www.trywaf.tk` in www.freenom.com

If you now access `www.trywaf.tk` because of above CNAME entry, traffic reaches to 
WAF node, WAF policy is applied and then it reaches to your origin i.e. OCI instance.

Access OWASP juice shop application. This application purposefully has many security 
vulnerabilities. Attack certain selected security vulnerabilities and observe that 
application is successfully attacked.
  
## Step 3 - Setting WAF Rules



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


#### User Agent
After we have had some activity, Go to Logs and find a user Agent in a JSON file for a log.
Go back to access rules and add new rule
Name:Safari Browser
Rule Condition: User Agent is 
Value: User agent you found in Json file
Select Block


### Add Protection Rules to Policy
Go to Protection Rules under your policy.
Filter by Rule ID (type these numbers in)
`973335
933161
960335
981242
90004
90001
941140
941110
911100
950007`
Select All (Check Box next to ID)
Select Actions: Block

Can also change protection rule actions to change blocked webpages message and amount of violations to trigger a protection rules.

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

### Trigger the rules in a demo.

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
Enter `5;return(true);` somewhere in your application.

#### PHP Vulnerability
Enter `process.exit()` into a box somewhere in your application

#### JavaScript Challenge
In postman run a get request on the main URL 11 times, on the 11th it should trigger the Javascript Challenge

#### Captcha Challenge
In a browser go to URL/captcha and it should trigger the captcha challenge

### Grafana Plugin with WAF data
First: Set up [OCI with Grafana](https://blogs.oracle.com/cloudnative/data-source-grafana) 
Second: 
##  Troubleshooting
  
502 Error -  Is having trouble with WAF connecting to Origin
