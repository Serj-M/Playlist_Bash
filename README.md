# Playlist_Bash
Bash-cкрипт под Linux. Аудио сервер, который составляет аудио плэйлист, чередуя рекламные трэки с музыкальной подборкой по заданному расписанию и в правильном порядке.

0 sudo su -
1 sudo apt-get update
2  apt-get install gcc
3  wget http://lftp.yar.ru/ftp/lftp-4.4.13.tar.gz
4  tar xzf lftp-4.4.13.tar.gz
5  cd lftp-4.4.13
6  ./configure
7  make
8  make install
9  sudo apt-get update
10  sudo apt-get install lftp-4.4.13
11  cd /bin  //выбрать правильную дерикторию 
12  nano script_lftp_list

        #!/bin/bash    
	HOST="192.168."
	USER="user"
	PASS="1234"
	LCD="/home/user/Mvideo/music"
	RCD="Music/music"
	lftp -u $USER,$PASS -e "mirror --delete --only-newer --verbose $RCD $LCD --log=/home/user/Mvideo/lftp_music.log ; bye;" $HOST

	LCD="/home/user/Mvideo/reklam"
	RCD="Music/reklam"
	lftp -u $USER,$PASS -e "mirror --delete --only-newer --verbose $RCD $LCD --log=/home/user/Mvideo/lftp_reklam.log ; bye;" $HOST

	# начало формирования playlist.m3u
	find /home/user/Mvideo/music/ -type f -name "*.mp3" -print > /home/user/Mvideo/music.m3u
	find /home/user/Mvideo/reklam/ -type f -name "*.mp3" -print > /home/user/Mvideo/reklam.m3u
	find /home/user/Mvideo/music/ -type f -name "*.MP3" -print >> /home/user/Mvideo/music.m3u
	find /home/user/Mvideo/reklam/ -type f -name "*.MP3" -print >> /home/user/Mvideo/reklam.m3u
	cp /dev/null /home/user/Mvideo/playlist.m3u
	cp /dev/null /home/user/Mvideo/playlist.log
	kol_str_rekl=`wc -l /home/user/Mvideo/reklam.m3u | awk '{print $1}'`
	n_str=1
	cat /home/user/Mvideo/music.m3u| while read line; do                     
	   echo "$line" >> /home/user/Mvideo/playlist.m3u
	   head -n "$n_str" /home/user/Mvideo/reklam.m3u | tail -n 1 >> /home/user/Mvideo/playlist.m3u
           if [ "$n_str" = "$kol_str_rekl" ]; then
               echo "обнуление счетчика рекламных роликов: , "$n_str"">>/home/user/Mvideo/playlist.log
	       n_str=1
	   else
	       n_str=`expr $n_str + 1`
	       echo "в плайлист добавлен рекламный ролик: , "$n_str"">>/home/user/Mvideo/playlist.log
	   fi
        done
	 
13  chmod +x script_lftp_list
14  ./script_lftp_list
15  sudo apt-get install ntp
16  установка vlc
17  crontab -e
18  0 23 * * * sh /home/user/Mvideo/script_runtime_lftp

                                    #!/bin/bash
                                    # файлы HOUR и MIN не удалять!!!
                                    cp /dev/null /home/user/Mvideo/hour
                                    cp /dev/null /home/user/Mvideo/min
                                    echo $((`cat /dev/urandom | hexdump -n2 -d | sed -r 's/[^ ]+ +0*//; q'` % 5))>>/home/user/Mvideo/hour
                                    echo $((`cat /dev/urandom | hexdump -n2 -d | sed -r 's/[^ ]+ +0*//; q'` % 59))>>/home/user/Mvideo/min
                                    HOUR=`cat /home/user/Mvideo/hour`
                                    MIN=`cat /home/user/Mvideo/min`
                                    for i in `atq | awk '{print $1}'`;do atrm $i;done
                                    at "$HOUR":"$MIN" -f /home/user/Mvideo/script_lftp_list
  
