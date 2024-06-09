# Домашнее задание к занятию "`Отказоустойчивость в облаке`" - `Солдатенкова Елизавета`

---

### Задание 1

```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

variable "yandex_cloud_token" {
  type        = string
  description = "Данная переменная потребует ввести секретный токен в консоли при запуске terraform plan/apply"
}

provider "yandex" {
  token     = var.yandex_cloud_token #секретные данные должны быть в сохранности!! Никогда не выкладывайте токен в публичный доступ.
  cloud_id  = "b1gpdshc41uidakoi3eh"
  folder_id = "b1gkl976h26uofgttn6s"
  zone      = "ru-central1-a"
}

resource "yandex_compute_instance" "vm" {
  count = 2
  name = "vm${count.index}"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd87kbts7j40q5b9rpjr"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }

  metadata = {
    user-data = "${file("./meta.txt")}"
  }

}
resource "yandex_vpc_network" "network-1" {
  name = "network1"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet-1"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}

resource "yandex_lb_target_group" "hw-1" {
  name = "hw-1"
  target {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    address = yandex_compute_instance.vm[0].network_interface.0.ip_address
  }
  target {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    address = yandex_compute_instance.vm[1].network_interface.0.ip_address
  }
}

resource "yandex_lb_network_load_balancer" "lb-1" {
  name = "lb-1"
  deletion_protection = "false"
  listener {
    name = "my-lb1"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
  }
  attached_target_group {
    target_group_id = yandex_lb_target_group.hw-1.id
    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/"
      }
    }
  }
}
```

![Balancer](https://github.com/lizaMosiyash/sflt-04/blob/master/screenshots/02.PNG)
![home page of balancer](https://github.com/lizaMosiyash/sflt-04/blob/master/screenshots/03.PNG)

