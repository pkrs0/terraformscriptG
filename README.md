resource "azurerm_resource_group" "RG1" {
  name = "rg1"
  location = "eastus"
}
resource "azurerm_virtual_network" "R1" {
  name                = "newvnet"
  address_space       = ["10.1.0.0/24"]
  location            = azurerm_resource_group.RG1.location
  resource_group_name = azurerm_resource_group.RG1.name
}

resource "azurerm_subnet" "Sub1" {
  name                 = "sub1"
  resource_group_name  = azurerm_resource_group.RG1.name
  virtual_network_name = azurerm_virtual_network.R1.name
  address_prefixes     = ["10.1.0.0/26"]
}

resource "azurerm_network_interface" "NIC1" {
  name                = "N1"
  location            = azurerm_resource_group.RG1.location
  resource_group_name = azurerm_resource_group.RG1.name

  ip_configuration {
    name                          = "myip"
    subnet_id                     = azurerm_subnet.Sub1.id
    private_ip_address_allocation = "Dynamic"
  }
}
resource "azurerm_windows_virtual_machine" "VM1" {
  name                = "VM1"
  resource_group_name = azurerm_resource_group.RG1.name
  location            = azurerm_resource_group.RG1.location
  size                = "Standard_F2"
  admin_username      = "vm"
  admin_password      = "Godisgreat@1"
  network_interface_ids = [
    azurerm_network_interface.NIC1.id,
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2016-Datacenter"
    version   = "latest"
  }
}
