#### Tutorial Video ####

https://www.youtube.com/watch?v=-sU0O82fdZs&list=PL7iMyoQPMtAP7XeXabzWnNKGkCex1C_3C&index=1

#### Official link to install Hashicorp Vault ###
https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install

### Once vault will install we can start it in "DEV" mode for testing and learning that stores credentials in memory that is not recommened for production, but we can 
start vault in SERVER mode that for production purpose that stores credentials in disk some where.

# Firstly we will try this in "DEV" mode and follow the commands as follows
$vault server -dev # It will give you vault address and vault token that need to be exported

# You can use this command as well to start the server in dev mode and to see ui of dev mode
$vault server -dev -dev-listen-address="0.0.0.0:8200"

# Now open a new terminal of the vault server and export these two things
$export VAULT_ADDR="http://127.0.0.1:8200" or  $export VAULT_ADDR="http://0.0.0.0:8200"
$export VAULT_TOKEN="oasdofi09808sdaf8sd09f8a0sd8f0a"

# Verify vault status by using this command
$vault status

# Now we can do read/write/delete operation for our credentials, but to that we need to enable the path for example I have a path like "my/secret" then we have to enable it 
first and then store the credentials, "KV" stands for key and value

$vault secrets enable -path=my kv

# Now store the credentials in the form of key and value
$vault kv put my/secret key-1=value-1 # to write or store the credentials
$vault kv get my/secret # to read or see the credentials
$vault kv delet my/secret # to delete or remove the credentials

# If you want to see the credentials in json format
$vault kv get -format=json my/secret

# To see the secret list or active secret engines
$vault secrets list # you will see some default secret engines as well don't delete those

# If you want to enable cloud provider secret engines then you can enable like this
$vault secrets enable -path=aws aws

# Configure the AWS secrets engine with IAM credentials and region, this user will have root permissions that can create other user/role credentials by it self
$vault write aws/config/root \
>access_key=YOUR_AWS_ACCESS_KEY \
>secret_key=YOUR_AWS_SECRET_KEY \
>region=us-west-2

# Now we will create a role and we will allow some permissions to that role or we can define it as default so it will have same permissions 
like access key and secret key or we can give similar format like jason policy format like aws 
that it can perform some actions like ec2 full access, and we will generate dynamic secret 
for that role whih will have a limited time span that will automatically change

##### Example-1 ######
$vault write aws/roles/prashant-vault-role \
>role_arn=arn:aws:iam::ACCOUNT_ID:role/YourIAMRole \
>policies=default

##### Example-2 ######
$vault write aws/roles/prashant-vault-role2 \
        credential_type=iam_user \
        policy_document=-EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1426528957000",
      "Effect": "Allow",
      "Action": [
        "ec2:*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
EOF




# Now to generate the dynamic access key & secret key for this role will have a limited life span and this time span is called as "TTL" or "Time to leave",
 we have to execute this command, Note pls provide the exact role name without any mistake

$vault read aws/creds/prashant-vault-role


# Revoke/Deactivate the secrets if you do not want it any longer, here you have to give the "lease_id" that was generated earlier
$ vault lease revoke aws/creds/my-ec2-role/J8WHZJ5NItdH23KYYHdORv3K



# To disable the cloud provider secret engine
-vault secrets disable aws




##### hashicorp vault starting in production environment to see GUI interface and stor credentials visually #####
https://www.youtube.com/watch?v=OyL4NpM_BPA

1>Vault must be installed on your machine. If you have started vault server in Dev mode then execute these two commands to start in Server/Production mode.

# To stop vault in dev mode
$ctrl + c

2> Unset vault token
$unset VAULT_TOKEN


3> Create a path where your all secrets will be store 
$mkdir -p vault/data

4> Provide all the details like ssl/tls certificate as well in this, If you have or keep it as it is
$vi config.hcl
paste this inside it 
-------------------------------------------------------------
ui = true
disable_mlock = true

storage "raft" {
  path    = "./vault/data"
  node_id = "node1"
}

listener "tcp" {
  address     = "0.0.0.0:8200" 
  tls_disable = "true"
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"
-------------------------------------------------------------

5>  Execute this command to start in production mode
$vault server -config=config.hcl  


6> Export the address 
$export VAULT_ADDR="http://127.0.0.1:8200"


7> Initialise Operator
$vault operator init


8> Now go to the UI with the public_ip and port 8200 and give unseal key and token
$vault operator unseal ## for CLI purpose you can use this command 


9> Rest all the steps and concept are same 
#### VVVi  ####
You can not create role or policy from GUI of the vault server, you have to use the CLI only for this
