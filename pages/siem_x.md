# Création de l'infrastructure de notre stack SIEM

## Initiation à Terraform et Ansible

Je compte profiter de ce projet pour me familiariser avec diverses technologies d'orcherstration, histoire de faire d'une pierre deux coups. Voyons ensemble les deux technologies utilisées dans le cadre de ce projet : 

### Terraform

Terraform est un outil open source d'IaC (Infrastructure as Code) publié par HashiCorp. Le concept d'IaC est plutôt explicite : toute mon infrastructure est déclarée dans des fichiers de configuration (en .tf) plutôt que mise en place manuellement. Il suffit ensuite de lancer quelques commandes dans le dossier `terraform` de notre environnement Cloud (Azure, AWS, etc) pour que l'infrastructure en question s'initialise. Ci-dessous un exemple de syntaxe Terraform (utilisé pour créer une machine virtuelle) : 

``` 
resource "azurerm_windows_virtual_machine" "ForLab-vm-dc" {
  name                     = "ForLab-vm-dc"
  computer_name            = var.dc-hostname
  size                     = var.dc-size
  provision_vm_agent       = true
  enable_automatic_updates = true
  resource_group_name      = data.azurerm_resource_group.ForLab-rg.name
  location                 = data.azurerm_resource_group.ForLab-rg.location
  timezone                 = var.timezone
  admin_username           = var.windows-user
  admin_password           = random_string.windowspass.result
  custom_data              = local.custom_data_content
  network_interface_ids    = [
    azurerm_network_interface.ForLab-vm-dc-nic.id,
  ]

  os_disk {
    name                 = "ForLab-vm-dc-osdisk"
    caching              = "ReadWrite"
    storage_account_type = "StandardSSD_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2016-Datacenter"
    version   = "latest"
  }

[...]

}
```


### Ansible

Ansible, de son côté, est une plateforme d'automatisation de déploiements/configuration/gestion via des fichiers de configurations appelées playbook (en .yml) décrivant étape par étape les différentes tâches devant être exécutées sur les clients ciblés. L'avantage d'Ansible est qu'il ne nécessite pas d'agent installé sur les périphériques du SI à installer, fonctionnant plutôt via SSH pour les clients Linux, et WinRM/LDAP/Kerberos pour les clients Windows. Ci-dessous un exemple de syntaxe Ansible, utilisé pour créer un domaine sur un DC précédemment créé : 

```yaml
# Install AD
- name: Create new AD domain
  win_domain:
    dns_domain_name: "{{ domain_name }}"
    safe_mode_password: "{{ ansible_password }}"
  register: domain_install

- name: Reboot after AD installation
  win_reboot:
  when: domain_install.reboot_required
```


### Notre cas de figure, rapidement résumé

En ce qui nous concerne, le github de notre projet `cloud_forensics` utilise ces deux technologies :

*	Terraform afin de provisioner mon infrastructure, notamment en créant mes différentes serveurs avec des images Ubuntu/Debian, mes clients avec des images Windows, et une machine d'attaque sous Debian.
*	Ansible est d'ailleurs installé sur ladite machine d'attaque Debian. Celle-ci, connectée au reste de l'infrastructure via SSH ou WinRM, utilisera des playbooks pré-écrits afin d'exécuter les tâches de post-provisionnement nécessaires sur les différentes machines.