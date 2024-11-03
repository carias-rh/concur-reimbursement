# aa-concur-reimbursement-internet

## Youtube video
Click to watch

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/vme0-oD118Y/0.jpg)](https://www.youtube.com/watch?v=vme0-oD118Y)

This work is based on the script created by @dmzoneill. It requires some python and selenium knowledge to adapt the invoice script to your particular services provider website, but it's easy and explained at the end of the readme.

It has been adapted for my needs and preferences of configuration, such using ansible-vault to protect secrets and HOTP to create the SSO RH login. 

I'm reusing some code from my previous work at `carias-rh/rol-lab-persistence` where I already had developed the logins and other RH related utility.

## setup.yml
By running this playbook you will have installed the required selenium python libraries, the oathtool utility and will output a random OTP secret that you will use to create your RH token.
``` 
$ ansible-playbook setup.yml -K
```

## generate-concur-report.yml
 - This playbook encapsulates all the functionality. 
 - It will generate 2 selenium scripts, one for downloading the invoice from your service provider and the other to create the concur report.
 - You may need to introduce your country as playbook variable as it appears in the concur report dropdown.
 - You will need to create a SSO OTP key at `token.redhat.com` to pass authentication.
 - see below for explanation of the individual python scripts and configuration.

### Usage
```
$ ansible-playbook generate-concur-report.yml
```
 

### SSO for Associates
As you may have seen, the `setup.yml` playbook generated a `secret` that we will use to create our SSO token. 
```

TASK [Use this secret string to create your token.redhat.com] *************************************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "generated_secret.stdout": "4439be1a......................bc2d3ec263"
}
...output omitted...
```

Fill in the `credentials.yml` file with your redhatter username and a PIN of your choice. Vault-encrypting this file is recommended:
```
# SSO Red Hat credentials
username: "rh-username"	                            # without @redhat.com
secret: "4439be1a......................bc2d3ec263"          # OTP Key to generate the SSO token
pin: "yourpin" 		                                    # Create a PIN for the OTP Key

# Service provider credentials
invoice_username: ""
invoice_password: ""
```


Go to `token.redhat.com` with the VPN activated to create the new token with the given secret. Uncheck the ☑️ `Generate OTP Key on the Server` box, paste your secret, and the chosen a PIN.

![image](https://user-images.githubusercontent.com/80515069/177427661-7a1d9c81-ad96-485c-a31a-376e7dc3c1e5.png)

Make sure that the `./counter` file always matches the `Count` value of the token, **initially set to 1**. It will increase the value each time you login.

![hotp](https://user-images.githubusercontent.com/80515069/212667043-69dd2e9e-c81e-4b75-a5ac-41e1b52b8f27.png)



## invoice.py

- This script needs to be adapted to your particular needs.
- It will login into the service provider site, download the latest bill and place it into this directory.
- The pdf file will be renamed with the current date in **Spanish format**. Adjust this to your needs by editing the main playbook that gathers the date.

### Usage
In case that you want to only run the invoice script use the `--tag invoice` parameter.
```
ansible-playbook generate-concur-report.yml -t invoice
```

### concur.py
```
python3 concur.py \
    --bill_date $bill_date \
    --receipt $MONTH.pdf \
    --location "Cork, IRELAND" \
    --vendor "Virgin Media"
```
### Usage
In case that you want to only run the concur script use the `--tag concur` parameter. Make sure that the invoice is placed in this directory with the current date as name.
```
ansible-playbook generate-concur-report.yml -t concur
```

## Adapting the invoice script to your services provider website
