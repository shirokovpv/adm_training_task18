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
<img width="752" height="967" alt="image" src="https://github.com/user-attachments/assets/3ebc9bca-c786-4e49-bd23-0ba4082eae1a" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Плейбук выполняет следующие задачи:</span></p>
<p><span style="font-weight: 400;">- устанавливает на обе ВМ пакет <strong>borgbackup</strong>;</span></p>
<p><span style="font-weight: 400;">- создает на обеих ВМ пользователя <strong>borg</strong> с паролем <strong>Otus1234</strong>;</span></p>
<p><span style="font-weight: 400;">- на ВМ <strong>backup</strong> создает директорию <strong>/var/backup</strong>;</span></p>
<p><span style="font-weight: 400;">- на ВМ <strong>backup</strong> создает раздел на диске <strong>/dev/sdb</strong> с файловой системой ext4 и монтирует его в директорию <strong>/var/backup</strong>.</span></p>
<p><span style="font-weight: 400;">Плейбук также прикладываю сюда.</span></p>
<p><span style="font-weight: 400;">После запуска и выполнения команды <code>vagrant up</code> (может занять какое-то время, в случае ошибки можно запустить повторно <code>vagrant up</code> или <code>vagrant provision</code> в зависимости от этапа выполнения*) увидим в VirtualBox две машины:</span></p>
<img width="982" height="502" alt="image" src="https://github.com/user-attachments/assets/30ca0fcc-b6fb-4a52-9463-d643291260cc" />
<p><span style="font-weight: 400;"><em>*Иногда при выполнении почему-то зависает - возможно, из-за обходного решения. При зависшей ВМ с ошибкой UNREACHABLE можно ее удалить командой <code>vagrant destroy</code></em></span></p>
<p><span style="font-weight: 400;">Проверим настройки клиента:</span></p>
<p><span style="font-weight: 400;"><code>ssh borg@192.168.56.150</code></span></p>
<p><span style="font-weight: 400;"><code>pwd</code></span></p>
<p><span style="font-weight: 400;"><code>whoami</code></span></p>
<p><span style="font-weight: 400;"><code>apt list borgbackup</code></span></p>
<img width="826" height="647" alt="image" src="https://github.com/user-attachments/assets/a846f535-b1fc-4530-bfa3-baf976223b89" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Проверим настройки сервера:</span></p>
<p><span style="font-weight: 400;"><code>ssh borg@192.168.56.160</code></span></p>
<p><span style="font-weight: 400;"><code>pwd</code></span></p>
<p><span style="font-weight: 400;"><code>whoami</code></span></p>
<p><span style="font-weight: 400;"><code>apt list borgbackup</code></span></p>
<p><span style="font-weight: 400;"><code>lsblk</code></span></p>
<img width="826" height="847" alt="image" src="https://github.com/user-attachments/assets/f73e0869-aeae-4bb5-929e-239075f92191" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Пользователь есть, заходит, домашний каталог есть, пакет установлен, на сервере диск примонтирован куда надо. Стенд собран.</span></p>
<p><strong>2) Настраиваем бэкап</strong></p>
