# adm_training_task18
<h1 align="center">Занятие 18. Настраиваем бэкапы</h1>
<ol>
<li style="font-weight: 400;"><span style="font-weight: 400;">Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client. (Студент самостоятельно настраивает Vagrant)</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Настроить удаленный бэкап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:</span></li>
<ol>
<li style="font-weight: 400;"><span style="font-weight: 400;">директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB; (Студент самостоятельно настраивает)</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">репозиторий для резервных копий должен быть зашифрован ключом или паролем - на усмотрение студента;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">имя бэкапа должно содержать информацию о времени снятия бэкапа;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">глубина бэкапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех. Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на усмотрение студента;</span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">настроено логирование процесса бэкапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.</span></li>
</ol>
</ol>
<h3 class="western"><a name="_heading=h.df570rpzx1qg"></a><span style="font-family: Roboto, serif;"><span style="font-size: small;">Используемые ОС</span></span></h3>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Хостовая ОС Ubuntu 24.04 Desktop с установленным Vagrant 2.3.5. VirtualBox версия 7.0.16_Ubuntu r162802</span></span></p>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Выполнение</span></span></h3>
<p>******************************</p>
<p>Ввиду невозможности в нынешних реалиях воспользоваться стандартными репозиториями Vagrant (в РФ они сейчас не доступны), предложено обходное решение. Использован репозиторий, развернутый на&nbsp;<a href="https://vagrant.elab.pro/" rel="nofollow">https://vagrant.elab.pro/</a></p>
<p>Так как официальный пакет последней версии Vagrant также не доступен для скачивания, пакет взят оттуда же. Версия 2.3.5.</p>
<img width="411" height="88" alt="image" src="https://github.com/user-attachments/assets/7591c684-f543-4bf6-bfa8-3706a7648c97" />
<p>&nbsp;</p>
<p>Для подключения репозитория надо добавить в Vagrant-файл строку:</p>
<p><code>ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'</code></p>
<p>И изменить box_name и box_version (как в репозитории, если туда зайти).</p>
<img width="480" height="306" alt="image" src="https://github.com/user-attachments/assets/c34c2b46-a072-4edd-8ae6-6f0c8d26cde8" />
<p>******************************</p>
<p><strong>1) Собираем стенд для задания с использованием Vagrant и Ansible</strong></p>
<p><span style="font-weight: 400;">Создадим директорию <strong>task18</strong>, а в ней Vagrantfile для создания двух виртуальных машин:</span></p>
<p><span style="font-weight: 400;"><strong>backup</strong> IP <strong>192.168.56.160</strong> </span><span style="font-weight: 400;">Ubuntu 24.04</span></p>
<p><span style="font-weight: 400;"><strong>client</strong> IP <strong>192.168.56.150</strong> Ubuntu 24.04</span></p>
<img width="1349" height="910" alt="image" src="https://github.com/user-attachments/assets/442d8e23-b5cf-4f9a-9846-ec08ac06ffdd" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Файл создает две ВМ, назначает им IP и hostname, а также добавляет на машину <strong>backup</strong> второй диск <strong>/dev/sdb</strong> размером 4 Gb, после чего запускает ansible-плейбук с дальнейшими настройками. Vagrant-файл прикладываю сюда.</span></p>
<p><span style="font-weight: 400;">Создадим также плейбук playbook.yaml для настройки виртуальных машин:</span></p>
<img width="735" height="981" alt="image" src="https://github.com/user-attachments/assets/3eca63d5-fed6-489a-bc70-0521d86cb469" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Плейбук выполняет следующие задачи:</span></p>
<p><span style="font-weight: 400;">- устанавливает на обе ВМ пакет <strong>borgbackup</strong>;</span></p>
<p><span style="font-weight: 400;">- создает на обеих ВМ пользователя <strong>borg</strong> с паролем <strong>Otus1234</strong>;</span></p>
<p><span style="font-weight: 400;">- на ВМ <strong>backup</strong> создает директорию <strong>/var/backup</strong>;</span></p>
<p><span style="font-weight: 400;">- на ВМ <strong>backup</strong> создает раздел на диске <strong>/dev/sdb</strong> с файловой системой ext4 и монтирует его в директорию <strong>/var/backup</strong>.</span></p>
<p><span style="font-weight: 400;">Плейбук также прикладываю сюда.</span></p>
<p><span style="font-weight: 400;">После запуска и выполнения команды <code>vagrant up</code> увидим в VirtualBox две машины:</span></p>
<img width="976" height="508" alt="image" src="https://github.com/user-attachments/assets/96c4ae80-968e-4126-898e-ba53949fe055" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Проверим настройки клиента:</span></p>
<p><span style="font-weight: 400;"><code>ssh borg@192.168.56.150</code></span></p>
<p><span style="font-weight: 400;"><code>pwd</code></span></p>
<p><span style="font-weight: 400;"><code>whoami</code></span></p>
<p><span style="font-weight: 400;"><code>apt list borgbackup</code></span></p>
<img width="664" height="654" alt="image" src="https://github.com/user-attachments/assets/61ef2e6f-0505-445d-af05-ef4e9e42f939" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Проверим настройки сервера:</span></p>
<p><span style="font-weight: 400;"><code>ssh borg@192.168.56.160</code></span></p>
<p><span style="font-weight: 400;"><code>pwd</code></span></p>
<p><span style="font-weight: 400;"><code>whoami</code></span></p>
<p><span style="font-weight: 400;"><code>apt list borgbackup</code></span></p>
<p><span style="font-weight: 400;"><code>lsblk</code></span></p>
<img width="665" height="815" alt="image" src="https://github.com/user-attachments/assets/52cf01e6-e96b-420d-819f-beb11686b53d" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Пользователь есть, заходит, домашний каталог есть, пакет установлен, на сервере диск примонтирован куда надо. Стенд собран.</span></p>
<p><strong>2) Настраиваем бэкап</strong></p>
<p><span style="font-weight: 400;">На сервере <strong>backup</strong> назначаем на каталог <strong>/var/backup</strong> права пользователя <strong>borg</strong>:</span></p>
<p><span style="font-weight: 400;"><code>vagrant ssh backup</code></span></p>
<p><span style="font-weight: 400;"><code>sudo -i</code></span></p>
<p><span style="font-weight: 400;"><code>chown borg:borg /var/backup/</code></span></p>
<img width="665" height="424" alt="image" src="https://github.com/user-attachments/assets/a4b9f3d8-4516-42d2-8b3c-304df4def0d3" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Убедимся, что каталог <strong>/var/backup</strong> пуст. Если в нем что-то есть, нужно все удалить, иначе borg может выдать ошибку:</span></p>
<p><span style="font-weight: 400;"><code>cd /var/backup</code></span></p>
<p><span style="font-weight: 400;"><code>ll</code></span></p>
<img width="463" height="123" alt="image" src="https://github.com/user-attachments/assets/fc9d6792-7f47-456f-ac4d-0d4d97f0b7cd" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">На сервере <strong>backup</strong> создаем каталог <strong>~/.ssh/authorized_keys</strong> в каталоге <strong>/home/borg</strong></span></p>
<p><span style="font-weight: 400;"><code>su - borg</code></span></p>
<p><span style="font-weight: 400;"><code>mkdir .ssh</code></span></p>
<p><span style="font-weight: 400;"><code>touch .ssh/authorized_keys</code></span></p>
<p><span style="font-weight: 400;"><code>chmod 700 .ssh</code></span></p>
<p><span style="font-weight: 400;"><code>chmod 600 .ssh/authorized_keys</code></span></p>
<img width="463" height="123" alt="image" src="https://github.com/user-attachments/assets/d407682f-268f-42e3-89e0-3fa562c5349c" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">На ВМ <strong>client</strong> заходим под пользователем <strong>borg</strong> и генерируем ключ без пароля (на запросе пароля просто жмем Enter):</span></p>
<p><span style="font-weight: 400;"><code>vagrant ssh client</code></span></p>
<p><span style="font-weight: 400;"><code>su - borg</code></span></p>
<p><span style="font-weight: 400;"><code>ssh-keygen</code></span></p>
<img width="810" height="416" alt="image" src="https://github.com/user-attachments/assets/fdc937ed-a55f-42dc-b7b9-ba17f3f885df" />
<p><span style="font-weight: 400;">*Overwrite (y/n)? y - спрашивает, потому что сначала сгенерировал ключ с паролем, а нам так не надо.</span></p>
<p><span style="font-weight: 400;">Копируем открытый ключ на сервер <strong>backup</strong></span></p>
<p><span style="font-weight: 400;"><code>ssh-copy-id borg@192.168.56.160</code></span></p>
<img width="810" height="340" alt="image" src="https://github.com/user-attachments/assets/c6203213-7da5-41f8-b36d-8ca5889b0a36" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Проверяем доступ на сервер <strong>backup</strong></span></p>
<p><span style="font-weight: 400;"><code>ssh borg@192.168.56.160</code></span></p>
<img width="647" height="387" alt="image" src="https://github.com/user-attachments/assets/2cdcddc8-5f91-4065-8697-215b66b04a08" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Зашли. Пароль не запрашивает, это нам понадобится для запуска автоматического задания.</span></p>
<p><span style="font-weight: 400;">Все дальнейшие действия будут проходить на ВМ <strong>client</strong>. Инициализируем репозиторий borg на сервере с клиента (ставим пароль <strong>Otus1234</strong>):</span></p>
<p><span style="font-weight: 400;"><code>borg init --encryption=repokey borg@192.168.56.160:/var/backup/</code></span></p>
<img width="805" height="405" alt="image" src="https://github.com/user-attachments/assets/3edbe9ce-050b-4011-8b39-ecc2dc565b77" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Запускаем для проверки создания бэкапа:</span></p>
<p><span style="font-weight: 400;"><code>borg create --stats --list borg@192.168.56.160:/var/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc</code></span></p>
<img width="805" height="325" alt="image" src="https://github.com/user-attachments/assets/67180b34-3f54-480f-a93a-a8bc42a21bf9" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Смотрим, что у нас получилось:</span></p>
<p><span style="font-weight: 400;"><code>borg list borg@192.168.56.160:/var/backup/</code></span></p>
<img width="805" height="101" alt="image" src="https://github.com/user-attachments/assets/d810fbc5-4d79-430d-bff8-992bc1fcf31e" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Смотрим список файлов в архиве:</span></p>
<p><span style="font-weight: 400;"><code>borg list borg@192.168.56.160:/var/backup/::etc-2025-11-09_13:54:31</code></span></p>
<img width="805" height="472" alt="image" src="https://github.com/user-attachments/assets/d4f172f0-e636-49ee-aaf7-bd1f8f2b7552" />
<p>......</p>
<img width="805" height="312" alt="image" src="https://github.com/user-attachments/assets/7ac98857-e662-4326-9c7b-a21bf0ad0e3f" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Восстановим файл из бэкапа:</span></p>
<p><span style="font-weight: 400;"><code>borg extract borg@192.168.56.160:/var/backup/::etc-2025-11-09_13:54:31 etc/hostname</code></span></p>
<img width="805" height="63" alt="image" src="https://github.com/user-attachments/assets/7f84106f-4d4d-455f-a358-7dbc59002fcb" />
<p><span style="font-weight: 400;">Файл восстановлен без ошибок.</span></p>
<p><strong>3) Автоматизируем создание бэкапов с помощью systemd</strong></p>
<p><span style="font-weight: 400;">Создаем сервис и таймер в каталоге <strong>/etc/systemd/system/</strong> из-под root-пользователя:</span></p>
<p><span style="font-weight: 400;"><code>nano /etc/systemd/system/borg-backup.service</code></span></p>




