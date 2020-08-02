
# 2FA_OTP_Openstack_Horizon
2FA_OTP login for Openstack Horizon
Clone from: https://github.com/mcaimi/openstack-horizon-2factor-auth

### Openstack Train

### Install qrcode lib:
```sudo pip install qrcode[pil]```
### Clone from github
```git clone [https://github.com/phamngocsonls/2FA_OTP_Openstack_Horizon]```
```cd 2FA_OTP_Openstack_Horizon/topt-lib```
### Copy lib
```sudo cp -r * /usr/lib/python2.7/```

  

### Install the TOTP Authentication Backend and Dashboard
* Openstack: 
```cp -rv 2fa_dashboard_plugin/totp /usr/share/openstack-dashboard/openstack_dashboard/dashboards/identity/```

* Devstack
```cp -rv 2fa_dashboard_plugin/totp```
```/opt/stack/horizon/openstack_dashboard/dashboards/identity/```

### Next the actual auth backend must be put in place:
* Openstack
```mkdir -p /usr/share/openstack-dashboard/openstack_dashboard/auth```
```cp -v 2fa_auth_plugin/* /usr/share/openstack-dashboard/openstack_dashboard/auth/```

* Devstack: 
```mkdir -p /opt/stack/horizon/openstack_dashboard/auth```
```devstack: cp -v 2fa_auth_plugin/* /opt/stack/horizon/openstack_dashboard/auth/```


### Configure Django Settings

On all horizon nodes, edit /usr/share/openstack-dashboard/openstack_dashboard/settings.py and set this parameters to change the authentication python class used by django:
Find and remove ```AUTHENTICATION_BACKENDS```
* Openstack

TOTP_DEBUG = False
TOTP_VALIDITY_PERIOD = 30
AUTHENTICATION_BACKENDS =('openstack_dashboard.auth.backend.TwoFactorAuthBackend',)


* Devstack:
```vi /opt/stack/horizon/openstack_dashboard/settings.py```
find AUTHENTICATION_BACKENDS and remove it.

  
### Enable the newly installed dashboard

Lastly, enable the dashboard:
* Openstack
```cp -v 2fa_dashboard_plugin/enabled/_3032_identity_totp_panel.py /usr/share/openstack-dashboard/openstack_dashboard/dashboards/enabled/```

  
* Devstack
```cp -v 2fa_dashboard_plugin/enabled/_3031_identity_totp_panel.py /opt/stack/horizon/openstack_dashboard/enabled/```

### Restart apache
```sudo systemctl restart apache2```

  
### OTP config:

- 1st, you must config mail for account (this is bug)

- Go to Idenity/Two Factor Authentication and click “Activate TOTP Token” to create new OTP. But otp qr image created in ./otp.ong (may be _opt_stack/horizon/opt.png). User can config at /2fa_dashboard_plugin/totp/tools.py

- Scan qrcode and sunmit token

- Logout and login with acc = acc ; password = password + otp



---------------------------
### From orgin repo (Openstack Queens and Later)
Fix Keystone policies to allow the token owner to update the user on keystone:


  * in /etc/openstack-dashboard/keystone_policy.json update the 'identity:update_user' policy to match this:

  "identity:update_user": "rule:cloud_admin or rule:admin_and_matching_target_user_domain_id or rule:owner",

  * create a file under /etc/keystone/policy.d called update_user.json and insert these lines inside:

  {
    "identity:update_user": "rule:cloud_admin or rule:admin_and_matching_target_user_domain_id or rule:owner"
  }

the previous line uses policy.v3cloudsample.json as a base template (see the official keystone GitHub repo for that).
