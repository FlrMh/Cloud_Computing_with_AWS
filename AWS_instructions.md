Creating a cloud EC2 instance(scalable cloud VM) for our local monolith(our app and database VMs):

!!! Note: Make sure our account always has `Ireland(eu-west-01)` as location.

1. Search EC2.

2. Select EC2 and `launch an instance`.

### Configuring our instance:

3. Name your instance. Best practice: name_group_ReasonForUse

- in my case: `florina-tech201-app`(because we are creating an instance first, for our `app` VM).

4. Select the OS needed. In my case Ubuntu 18.04.

5. Select the instance type. In my case, `t2.micro`.

6. Select a key pair for this instance. In my case, select the key I received to be able to access this instance and that I further stored within the `.ssh` folder on my local host. 

7. Network settings:

1. Allow SSH Traffic - Only my IP.
2. Allow HTTP traffic from the internet(HTTPS - mainly used in production, when the site needs to be secure).

3. Go to Edit( Create a security group so we can use it another time as well): 
- VPC - default
- Subnet - Default subnet recommended.
- Auto-assign public IP - Enable (if it is an instance that should not be accessible- Disable)
- Firewall - Create security group.
- Security group name - FLORINA-TECH201-APP(we create this firewall for our app VM only)
- Security group description - The same as the Security group name + any other important things we need to mention.
- Sec rule 1 - ssh- My IP
- sec rule 2  - HTTP - Anywhere
- sec rule 3 (must be added -  Add Security group rule) - Custom TCP - Anywhere - 3000 - For Node App


8. Configure storage:
- Currently default

9. Summary:
- Need to make sure it matches the configuration we just made. 

10. Launch the instance.

11. Click the ID to get to the dashboard and see the created instance.