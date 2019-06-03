# WAFCookbook

##  Prereqs
Have an OCI account (Cloud.oracle.com). 

Have access to Edge Services in OCI. 

Provision an instance in OCI region. 

**Install docker on this VM.**


`yum install docker-engine`

`systemctl start docker`

`systemctl enable docker`


Pull docker image of open source software ORAWASP Juice Shop application. Run this 
application on host port 80
Ensure that security list used by the instance has appropriate rule so that port 80 is 
accessible from internet

Register to website www.freenom.com to create domain (For ex.) www.trywaf.tk

Ensure that > DNS A record is created for your domain that resolve to public IP of OCI instance.

Create WAF policy in Oracle Edge service to protect the origin of this application i.e. the 
public IP of instance with default port 80


Note down the CNAME target of this WAF policy

Now make CNAME record in the DNS of your domain that maps to 
www.trywaf.tk

If you now access 
www.trywaf.tk because of above CNAME entry, traffic reaches to 
WAF node, WAF policy is applied and then it reaches to your origin i.e. OCI instance.

Access OWASP juice shop application. This application purposefully has many security 
vulnerabilities. Attack certain selected security vulnerabilities and observe that 
application is successfully attacked.

Enable WAF protection rules to this WAF policy.

Access OWASP juice shop application using WAF protected URL. Observe that 
application is now protected from that attack. This proves WAF policy is working to 
prevent such attack. When using URL with public IP application works without having 
protection from vulnerabilities

## Step 1- Launch Webapp and add it to a domain

  
## Step 2
  

## Step 3

##  Troubleshooting
  
  
  502 Error -  
