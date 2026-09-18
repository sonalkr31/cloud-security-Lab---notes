# IAM :- Identity Access Managent of aws

 IAM is service that helps us securely control access to AWS resorces
 It allows us to manage users ,roles , and permission to define who csn access what within your AWS environment.

 IAM
 * free service 
 * Global Service - in dashboard you will see in region (Global)
 * Root account created by default , shouldn't be  used or shared.

 ## What we can do in IAM service 
 * Create user :- we can create individual user account for people who need to access to our AWS resources 

 * Assign permission :- We can asssign specific permission to users , groups, roles to control what action they can perform on aws services.

 * Create groups :-  We can group users togerther and assign permission to the group , management easier for multiple users.
 * Create roles :- we can create roles tto assign temporary permission to AWS services or users , specially useful for securely managing permission across different aws resources.
 * Define policies  ;- We can Create and attach customs policies to define fine-grained permission for controlling access to AES resources.

 * Manage Federated Access:- IAM allows intergrating with external identity providrs (like active directory) for centralized management of user access across Aws.

 ## MFA - multi fsctor auhentication .

 is an extra layer  of security that requires users to provides two or more forms of verification , like password and a code from their phone , to access their accounts.

 ## Way of accesing AWS

 * our main website of aws (GUI mode).

 * with cloud shell its on top on navigation bar. by command mode (cli) 
 we can only get this cli mode when we have logged in our aws account or website , so we donot need any kind of id or password.

 * Local cli :- with our laptop terminal -
 ## Why we use cli  more

 * For maintaince task RePETITIVE task , we can automate , automation , so it will save our time and very useful

* SDK and APIs ; it offeres prigrammatic , code-based access , alloeing users to integrate AWS directly into their applications.

  EX- let us say we are develoopoing an appilication in ppython , java , javascripts etc which there is need of integration with aws .

  ## Setup cli in  windows , mac and others

  * Inn widows go to google and search aws cli and download the cli setps from aws . and install like noraml windows program 

  * in MAC : use the homebrew through cli 

    $ brew install awscli
 check- aws --version


 ## Function

  try this web page of aws to get full command of all options and things .

 Link :-   https://docs.aws.amazon.com/cli/v1/userguide/cli_code_examples.html

    first configure the terminal using your access key  and secure key .

    and type command in local cli 
     $ aws configure
        and provide you region like   eu-north-1

        now you can run  the cmd to check access in local cli

        $ aws iam list-users

# AWS IAM Best Practices
* Avoid using root account except of account setup.
* Add user to a group and assign permission to group
*  Use password policy or MFA
* Use ACCESS KEYS for CLI/SDK
*  Never share ACCESS KEYS or Password
*  Audit the permission using IAM credential report.


