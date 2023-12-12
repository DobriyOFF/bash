#!/bin/bash

DEPLISTPATH="https://web.aanda.ru/department.php?deps"
WGET='/usr/bin/wget'
WGETARGS="--quiet --no-check-certificate"
DEPARTMENTS="$($WGET $WGETARGS -O - "$DEPLISTPATH")" # список отделов
REQSOFT="curl samba nfs-kernel-server htop mc rsync nmap"
userPasswd="1_qawsed"

install_missing_packages() {
  for package in $REQSOFT; do
    if ! dpkg -l | grep -q $package; then
      echo "apt install $package"
    fi
  done
}

create_directory_if_not_exists() {
  local directory="$1"
  if ! [ -d "$directory" ]; then
    mkdir "$directory"
  fi
}

create_group_if_not_exists() {
  local group="$1"
  grep $group /etc/group >/dev/null || addgroup $group
}

set_permissions() {
  local directory="$1"
  local group="$2"
  chown -R root:$group "$directory"
  chmod -R 770 "$directory"
}

create_users_and_set_samba_passwords() {
  local department="$1"
  local dep_user_path="https://web.aanda.ru/department.php?deplist=$department"
  local users="$($WGET $WGETARGS -O - "$dep_user_path")"

  for user in $users; do
    if ! getent passwd "$user" >/dev/null; then
      useradd -d "/storage/users/$user" -g "$department" -s /bin/bash "$user"
      (echo "$userPasswd"; echo "$userPasswd") | smbpasswd -a "$user"
      echo -e "$userPasswd\n$userPasswd" | smbpasswd -a -s "$user"
      smbpasswd -e "$user"
    fi
  done
}

# Установка отсутствующих пакетов
install_missing_packages

# Создание директорий
create_directory_if_not_exists "/storage"
create_directory_if_not_exists "/storage/departments"
create_directory_if_not_exists "/storage/software"
create_directory_if_not_exists "/storage/drivers"
create_directory_if_not_exists "/storage/users"
create_directory_if_not_exists "/storage/sbis"

# Создание групп
for department in $DEPARTMENTS; do
  create_group_if_not_exists "$department"
done

# Установка прав доступа
for department in $DEPARTMENTS; do
  create_directory_if_not_exists "/storage/departments/$department"
  set_permissions "/storage/departments/$department" "$department"
done

# Создание пользователей и установка паролей Samba
for department in $DEPARTMENTS; do
  create_users_and_set_samba_passwords "$department"
done
