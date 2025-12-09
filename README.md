# ‍🚀 **EC2 Chef Automation Project (Complete Guide + Code)**

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20250825181947907154/Chef-Architecture.webp?utm_source=chatgpt.com)

![Image](https://docs.chef.io/images/automate/a2-architecture-os.png?utm_source=chatgpt.com)

![Image](https://docs.chef.io/images/server/server_components_postgresql_14.svg?utm_source=chatgpt.com)

![Image](https://docs.chef.io/images/chef_overview_2020.svg?utm_source=chatgpt.com)

---

# ✅ **1. Project Overview**

| Component                       | Purpose                                        |
| ------------------------------- | ---------------------------------------------- |
| **Chef Server**                 | Centralized configuration + policy store       |
| **Chef Workstation**            | Where cookbooks/recipes are developed          |
| **EC2 Node**                    | Managed system (client) bootstrapped with Chef |
| **Cookbook (apache-webserver)** | Installs + configures Apache                   |
| **Role (webserver)**            | Defines how EC2 behaves                        |
| **Environment (prod)**          | Policy settings, version pins                  |
| **Knife Bootstrap**             | Registers EC2 to Chef Server                   |
| **Test Kitchen**                | Local testing using Vagrant/Docker             |

---

# ✅ **2. Architecture Diagram**

```
Chef Workstation  ----- Upload Cookbooks ---> Chef Server
      |                                              |
      |---- knife bootstrap -------------------------|
                      |
                 EC2 Node (Chef Client)
                      |
           Converges Recipes & Roles
```

---

# ✅ **3. Prerequisites**

### On local machine:

* Install **Chef Workstation**
* Install AWS CLI + access keys
* GitHub repo created (example):

```
ec2-chef-automation
```

### On AWS:

* EC2 Amazon Linux 2 instance
* Security Group: Port 22, 80
* Key Pair

---

# ✅ **4. Project Folder Structure (GitHub-Ready)**

```
ec2-chef-project/
│── cookbooks/
│    └── apache-webserver/
│         ├── recipes/
│         │     ├── default.rb
│         │     ├── install.rb
│         │     ├── configure.rb
│         ├── templates/
│         │     └── index.html.erb
│         ├── metadata.rb
│         └── kitchen.yml
│
│── roles/
│    └── webserver.json
│── environments/
│    └── prod.json
│── knife.rb
│── README.md
```

---

# ⭐ **5. Create the Cookbook**

```
chef generate cookbook apache-webserver
```

---

# ⭐ **6. Recipe Code**

## 📌 **install.rb**

```ruby
package 'httpd' do
  action :install
end
```

## 📌 **configure.rb**

```ruby
service 'httpd' do
  action [:enable, :start]
end

template '/var/www/html/index.html' do
  source 'index.html.erb'
  mode '0644'
end
```

## 📌 **default.rb**

```ruby
include_recipe 'apache-webserver::install'
include_recipe 'apache-webserver::configure'
```

---

# ⭐ **7. Template File**

## `templates/index.html.erb`

```html
<h1>Welcome to Cloudnautic EC2 Server <%= node['hostname'] %></h1>
<h2>Automated via Chef!</h2>
```

---

# ⭐ **8. metadata.rb**

```ruby
name 'apache-webserver'
maintainer 'Atul'
version '1.0.0'
supports 'amazon'
```

---

# ⭐ **9. Create a Role**

## roles/webserver.json

```json
{
  "name": "webserver",
  "description": "Apache Web Server Role",
  "run_list": [
    "recipe[apache-webserver]"
  ]
}
```

Upload:

```
knife role from file roles/webserver.json
```

---

# ⭐ **10. Environment File**

## environments/prod.json

```json
{
  "name": "prod",
  "description": "Production EC2 Environment",
  "cookbook_versions": {},
  "default_attributes": {},
  "override_attributes": {}
}
```

Upload:

```
knife environment from file environments/prod.json
```

---

# ⭐ **11. Upload Cookbook to Chef Server**

```
knife cookbook upload apache-webserver
```

---

# ⭐ **12. Bootstrap EC2 Node**

### Syntax:

```
knife bootstrap <EC2-PUBLIC-IP> \
  --ssh-user ec2-user \
  --sudo \
  --identity-file /path/key.pem \
  --node-name ec2-web-01 \
  --run-list "role[webserver]"
```

Example:

```
knife bootstrap 3.111.25.89 \
--ssh-user ec2-user \
--sudo \
--identity-file ~/aws/atul.pem \
--node-name cloudnautic-web1 \
--run-list "role[webserver]" \
--environment prod
```

---

# ⭐ **13. Verify on Chef Server**

```
knife node list
knife node show cloudnautic-web1
```

---

# ⭐ **14. Check EC2 Web Output**

Browser:

```
http://EC2_PUBLIC_IP
```

You should see:

```
Welcome to Cloudnautic EC2 Server ip-XXX
Automated via Chef!
```

---

# ⭐ **15. Test Kitchen Configuration**

## `.kitchen.yml`

```yaml
driver:
  name: ec2
  region: ap-south-1
  instance_type: t2.micro
  image_id: ami-0f58b397bc5c1f2e8
  subnet_id: subnet-123456789

provisioner:
  name: chef_zero

verifier:
  name: inspec

platforms:
  - name: amazonlinux2

suites:
  - name: default
    run_list:
      - recipe[apache-webserver::default]
```

Commands:

```
kitchen create
kitchen converge
kitchen verify
kitchen destroy
```

---

# ⭐ **16. README.md (GitHub Template)**

I can generate full README.md if you want!

---

# ⭐ **17. Add AWS EC2 Creation with Knife (Optional)**

```
knife ec2 server create \
  --image ami-0f58b397bc5c1f2e8 \
  --flavor t2.micro \
  --region ap-south-1 \
  --security-group-ids sg-xxxx \
  --ssh-key atul \
  --node-name chef-ec2-node \
  --run-list "role[webserver]"
```

---

# 🎉 **Project Complete**

This includes:

✔ Full cookbook
✔ Recipes
✔ Templates
✔ Roles
✔ Environments
✔ EC2 bootstrap
✔ Test Kitchen
✔ GitHub structure
✔ Commands

---
