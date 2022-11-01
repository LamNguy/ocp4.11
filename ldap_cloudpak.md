# Tích hợp LDAP với IBM Cloud Pak for Watson AIOps

# I. Cấu hình tại phía Client server

## 1.1.  Verify certificate ở phía server ldaps://10.3.12.17:636

```bash
**$ openssl s_client -showcerts -connect 10.3.12.17:636**
// output omitted

-----BEGIN CERTIFICATE-----
...........................
...........................
-----END CERTIFICATE-----
```

## 1.2. Thêm certificate cho client

```bash
**$ mkdir -p /etc/openldap/cacerts
$ vim /etc/openldap/cacerts/certificate.pem**
// output omitted

-----BEGIN CERTIFICATE-----
...........................
...........................
-----END CERTIFICATE-----
```

## 1.3. Cấu hình Openldap for test

```bash
**$ vi /etc/openldap/ldap.conf**

SASL_NOCANON    on
URI             ldaps://10.3.12.17:636
BASE            ou=QLVAS,ou=NOC,dc=mobifone,dc=vn

# do certificate mobifone bị vấn đề ca root lên ignore verify ca
TLS_REQCERT     allow
TLS_CACERTDIR   /etc/openldap/cacerts
TLS_CACERT      /etc/openldap/cacerts/certificate.pem
```

## 1.4. Verify kết nối tới ldap server sử dụng ldapsearch

```bash
**$ ldapsearch -x -LLL -H ldap://10.3.12.17:389 -D privatecloud -w n8Z10Yn28# -b "ou=QLVAS,ou=NOC,dc=mobifone,dc=vn" -s sub "(givenName=privatecloud)"**
```

```bash
# result
dn: CN=privatecloud,OU=QLVAS,OU=NOC,DC=mobifone,DC=vn
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: privatecloud
description: Truong Trung Trang 23032022
physicalDeliveryOfficeName: TT.NOC
telephoneNumber: +84905666096
givenName: privatecloud
distinguishedName: CN=privatecloud,OU=QLVAS,OU=NOC,DC=mobifone,DC=vn
instanceType: 4
whenCreated: 20220323080558.0Z
whenChanged: 20221031083439.0Z
displayName: privatecloud
uSNCreated: 35037291
memberOf: CN=privatecloudnoc,OU=QLVAS,OU=NOC,DC=mobifone,DC=vn
uSNChanged: 157301282
company: Trung tam Quan ly, Dieu hanh mang
name: privatecloud
objectGUID:: LgC/00R0kUqi8PWSJKTE8g==
..........................
output omitted
```

## 1.5. Cấu hình SSSD

 - lưu ý: SSSD must be configured to communicate with LDAP over SSL or Start TLS

```bash
# step 1: cấu hình sssd với autoselect
**$ authselect select sssd --force**
```

```bash
# step 2: auto select not automatically generate sssd.conf, need create
$ TEMPLATE_DIR="/opt/IBM/infrastructure-management-appliance/TEMPLATE"
$ mkdir -p /etc/sssd
$ cp ${TEMPLATE_DIR}/etc/sssd/sssd.conf.erb /etc/sssd/sssd.conf
$ chmod 600 /etc/sssd/sssd.conf

```

```bash
# step 3: cau hinh /etc/sssd/sssd.conf
[domain/default]
ldap_search_base = dc=mobfone,dc=vn
ldap_uri = ldaps://10.3.12.17:636
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_id_use_start_tls = False
cache_credentials = True

[domain/mobifone.vn]
#debug_level = 9
ldap_id_mapping = true
enumerate = true
autofs_provider = ldap
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
#ldap_schema = rfc2307bis
ldap_schema = AD
ldap_uri = ldaps://10.3.12.17:636
ldap_tls_reqcert = allow
ldap_id_use_start_tls = False
ldap_tls_cacertdir = /etc/openldap/cacerts
ldap_tls_cacert = /etc/openldap/cacerts/certificate.pem
ldap_pwd_policy = none

ldap_search_base = ou=QLVAS,ou=NOC,dc=mobifone,dc=vn
ldap_network_timeout = 3

# search base cho user
ldap_user_search_base = ou=QLVAS,ou=NOC,dc=mobifone,dc=vn

# filter user co object class la user
ldap_user_object_class = user
ldap_default_bind_dn = cn=privatecloud,ou=QLVAS,ou=NOC,dc=mobifone,dc=vn
ldap_default_authtok_type = password
ldap_default_authtok = n8Z10Yn28#

# mapping ldap **user_name** voi truong **giveName** or **displayName** voi ket qua o buoc 4
ldap_user_name = givenName

# mapping **uid_number** voi truong **name** voi ket qua buoc 4
ldap_user_uid_number = name

# maping **gid_number** voi truong **primaryGroupID** voi ket qua buoc 4
ldap_user_gid_number = primaryGroupID

# filter class cua user **privatecloud** co thuoc tinh class la ***group***
ldap_group_object_class = group

# search base cho group
ldap_group_search_base = cn=privatecloudnoc,ou=QLVAS,ou=NOC,dc=mobifone,dc=vn
ldap_group_name = name
ldap_group_member = member

cache_credentials = True
entry_cache_timeout = 600
# cau hinh cac thuoc tinh them de test, cho nay chua clear lam
ldap_user_extra_attrs = givenName, displayName

[sssd]
domains = mobifone.vn
services = nss, pam, ifp
config_file_version = 2
sbus_timeout = 100
default_domain_suffix = mobifone.vn
debug_level = 9

[nss]
debug_level = 9
homedir_substring = /home

[pam]
debug_level = 9
default_domain_suffix = mobifone.vn

[sudo]

[autofs]

[ssh]

[pac]

[ifp]
debug_level = 9
default_domain_suffix = mobifone.vn
allowed_uids = privatecloud,apache,root
user_attributes = +givenName, +displayName

[session_recording]
```

## 1.6. Khởi động SSSD và verify

```bash
# verify file cau hinh, output giong nhu nay thi OK
**$ sssctl config-check**
*Issues identified by validators: 3
[rule/allowed_sssd_options]: Attribute 'sbus_timeout' is not allowed in section 'sssd'. Check for typos. 
[rule/allowed_pam_options]: Attribute 'default_domain_suffix' is not allowed in section 'pam'. Check for typos.
[rule/allowed_ifp_options]: Attribute 'default_domain_suffix' is not allowed in section 'ifp'. Check for typos.
Messages generated during configuration merging: 0

Used configuration snippet files: 0*

# start dich vu ssd
**$ systemctl start sssd**

# kiem tra user private cloud, neu connect thanh cong sssd voi ldap server thi query dc ket qua
****$ **id privatecloud**
uid=1057716196(privatecloud@mobifone.vn) gid=1057600513 groups=1057600513

# kiem tra danh sach domain, expect co domain mobifone.vn
**$ sssctl domain-list**
implicit_files
mobifone.vn

# kiem tra trang thai domain mobifone.vn, exptect status la online
**$ sssctl domain-status mobifone.vn**
Online status: **Online**

Active servers:
LDAP: 10.3.12.17

Discovered LDAP servers:
- 10.3.12.17

# kiem tra trang thai user privatecloud, dam bao co hien ra 2 truong extra attribute la givenName va displayName 
**$ sssctl user-checks privatecloud**
user: privatecloud
action: acct
service: system-auth

SSSD nss user lookup result:
 - user name: privatecloud@mobifone.vn
 - user id: 1057716196
 - group id: 1057600513
 - gecos: privatecloud
 - home directory:
 - shell:

SSSD InfoPipe user lookup result:
 - name: privatecloud@mobifone.vn
 - uidNumber: 1057716196
 - gidNumber: 1057600513
 - gecos: privatecloud
 - homeDirectory: not set
 - loginShell: not set
 - displayName: privatecloud
 - givenName: privatecloud

testing pam_acct_mgmt

pam_acct_mgmt: Success

# verify su dung dbus
**$ setenforce 0**
$ **dbus-send --print-reply --system --dest=org.freedesktop.sssd.infopipe /org/freedesktop/sssd/infopipe org.freedesktop.sssd.infopipe.GetUserAttr string:privatecloud array:string:displayName,givenName**

// output like that
method return time=1667277809.367565 sender=:1.238 -> destination=:1.239 serial=6 reply_serial=2
   array [
      dict entry(
         string "displayName"
         variant             array [
               string "privatecloud"
            ]
      )
      dict entry(
         string "givenName"
         variant             array [
               string "privatecloud"
            ]
      )
   ]

**# verify group, thông tn group naày dunùng để taạo** 
$ dbus-send --print-reply --system --dest=org.freedesktop.sssd.infopipe /org/freedesktop/sssd/infopipe org.freedesktop.sssd.infopipe.GetUserGroups string:privatecloud

method return time=1667277869.817209 sender=:1.238 -> destination=:1.240 serial=8 reply_serial=2
   array [
      string "privatecloudnoc@mobifone.vn"
      string "privatecloudnoc@mobifone.vn"
   ]

```

### *Trường hợp xảy ra lỗi, check các log này để lấy thêm thông tin*

| Log File | Type of debugging |
| --- | --- |
| /var/log/sssd/sssd.log | SSSD communication with processes |
| /var/log/sssd/sssd_example.com.log | sssd-ldap communication to the LDAP server |
| /var/log/sssd/sssd_ifp.log | Gathering user and group information from LDAP server |
| /var/log/message |  |
| /var/log/secure |  |

Cấu hình debug level 9 cho các trường thông tin như ifp, domain, sssd để kiểm tra log

```bash
# /etc/sssd/sssd.conf
[ifp]
==> debug_level = 9
default_domain_suffix = mobifone.vn
```

Clear cache sau khi verify xong

```bash
$ sss_cache -E
```

## 1.7. Cấu hình Apache

```bash
# cau hinh apache
$ TEMPLATE_DIR="/opt/IBM/infrastructure-management-appliance/TEMPLATE"
$ cp ${TEMPLATE_DIR}/etc/pam.d/httpd-auth                         \
                    /etc/pam.d/httpd-auth
$ cp ${TEMPLATE_DIR}/etc/httpd/conf.d/manageiq-remote-user.conf       \
                    /etc/httpd/conf.d/
$ cp ${TEMPLATE_DIR}/etc/httpd/conf.d/manageiq-external-auth.conf.erb \
                    /etc/httpd/conf.d/manageiq-external-auth.conf
```

## 1.8. Cấu hình selinux

### *Trường hợp port LDAP server chạy khác 2 port 389 và 636 thì phải thực hiện allow selinux*

```bash
$ semanage port -a -t ldap_port_t -p tcp 10389
$ semanage port -a -t ldap_port_t -p tcp 10636
```

### *Cấu hình selinux permission*

```bash
$ setsebool -P allow_httpd_mod_auth_pam on
$ setsebool -P httpd_dbus_sssd          on
```

## 1.9 Khởi động lại dịch vụ

```bash
$ systemctl restart sssd
$ systemctl restart httpd
```

# II. **Configure Administrative UI**

## 2.1 Login vào applicance

Log in as admin, navigate to *Settings > Application Settings > Settings,*  Select the server, then select the *Authentication tab*

- Set mode to External (httpd)
- Check: *Get User Groups from External Authentication (httpd)*
- Do not check: *Enable Single Signon* since Kerberos is not configured against LDAP.
- Click Save.

The above steps need to be completed on each UI and WebService enabled appliance.

Navigate to *Settings > Application Settings > Access Control*

## 2.2 Make sure the user’s LDAP group for the appliance are created and appropriate roles assigned to those groups.

- Tạo role ldapuser phù hợp
- Tạo group ldapgroup có tên **privatecloudnoc** trùng với tên ldap group ở phía server
- Gắn role ldapuser đã tạo

## 2.3 Logout và đăng nhập lại với account privatecloud

- không cần tạo user từ app, khi đăng nhập thì user này được tự tạo và gắn vào group **privatecloudnoc**
