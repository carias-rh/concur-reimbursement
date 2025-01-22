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
