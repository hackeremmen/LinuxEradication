# LinuxEradication
Linux Eradication



										TryHackMe - Advent Of Cyber 2023 | Eradication
										==============================================


Learning Objectives
-------------------


Bu vəzifədə biz:

Linux-da CPU və proseslərin yaddaş istifadəsini müəyyən edin.
Linux-də arzuolunmaz prosesləri öldürün.
Bir prosesin sona çatmadan davam edə biləcəyi yolları tapın.
Davamlı prosesləri daimi olaraq aradan qaldırın.

Identifying the Process
-----------------------

Linux bizə sistemin performansını izləmək üçün müxtəlif seçimlər təqdim edir. Bunlardan istifadə etməklə biz proseslərin resurs istifadəsini müəyyən edə bilərik. Seçimlərdən biri top əmridir. Bu əmr bizə real vaxt rejimində onların istifadəsi ilə proseslərin siyahısını göstərir. Bu, dinamik siyahıdır, yəni hər bir prosesin resurs istifadəsi ilə dəyişir.


Gəlin bu əmri əlavə edilmiş VM-də işlətməklə başlayaq. Biz terminalda top yazıb enter düyməsini basa bilərik. O, aşağıdakı nəticəyə oxşar çıxışı göstərəcək:

ubuntu@tryhackme:~$ top
top - 03:40:19 up 32 min,  0 users,  load average: 1.02, 1.08, 1.11
Tasks: 187 total,   2 running, 183 sleeping,   0 stopped,   2 zombie
%Cpu(s): 50.7 us,  0.3 sy,  0.0 ni, 48.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.2 st
MiB Mem :   3933.8 total,   2111.3 free,    619.7 used,   1202.8 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   3000.4 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND   
   2062 root      20   0    2488   1532   1440 R 100.0   0.0  18:22.15 a         
    941 ubuntu    20   0  339800 116280  57168 S   1.0   2.9   0:08.27 Xtigervnc 
   1965 root      20   0  123408  27700   7844 S   1.0   0.7   0:02.83 python3   
   1179 lightdm   20   0  565972  44756  37252 S   0.3   1.1   0:02.25 slick-gr+ 
   1261 ubuntu    20   0 1073796  38692  30588 S   0.3   1.0   0:01.10 mate-set+ 
      1 root      20   0  104360  12052   8596 S   0.0   0.3   0:04.52 systemd   
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd  
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp    
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_+ 
      5 root      20   0       0      0      0 I   0.0   0.0   0:00.43 kworker/+ 
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/+ 
      9 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 mm_percp+ 
     10 root      20   0       0      0      0 S   0.0   0.0   0:00.12 ksoftirq+ 
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.50 rcu_sched 
     12 root      rt   0       0      0      0 S   0.0   0.0   0:00.01 migratio+ 
     13 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/0   
     14 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/1   
     15 root      rt   0       0      0      0 S   0.0   0.0   0:00.31 migratio+ 
     16 root      20   0       0      0      0 S   0.0   0.0   0:00.13 ksoftirq+ 
     18 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/+ 
     19 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kdevtmpfs 
     20 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 netns     
     21 root      20   0       0      0      0 S   0.0   0.0   0:00.00 rcu_task+ 
     22 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kauditd   
     23 root      20   0       0      0      0 S   0.0   0.0   0:00.00 xenbus    
     24 root      20   0       0      0      0 S   0.0   0.0   0:00.03 xenwatch  
     25 root      20   0       0      0      0 S   0.0   0.0   0:00.00 khungtas+ 
     26 root      20   0       0      0      0 S   0.0   0.0   0:00.00 oom_reap+ 


Killing the Culprit
-------------------

top əmrinin çıxışının yuxarı hissəsində günahkarımızı tapırıq. Bu, qeyri-adi yüksək CPU resurslarından istifadə edən a adlı prosesdir. Normal şəraitdə çox yüksək miqdarda CPU resursundan istifadə edən proseslərə davamlı olaraq sahib olmamalıyıq. Bununla belə, müəyyən proseslər intensiv emal üçün bunu qısa müddətə edə bilər.

Burada gördüyümüz proses ardıcıl olaraq CPU resurslarının 100%-ni istifadə edir ki, bu da kriptomoner kimi zəhmətkeş bir zərərli prosesi ifadə edə bilər. . Kök istifadəçinin bu prosesi işlətdiyini görürük. Proses' ad və resurs istifadəsi şübhəli əhval-ruhiyyə verir və bunun bizim resurslarımızı lüzumsuz şəkildə ələ keçirən proses olduğunu fərz etsək, onu öldürmək istərdik. (İmtina: Faktiki istehsal serverlərində, nə etdiyinizə əmin deyilsinizsə, prosesləri öldürməyə çalışmayın.)

Əgər məhkəmə ekspertizası aparmaq istəsək, onu öldürməzdən əvvəl onu daha ətraflı təhlil etmək üçün yaddaş zibilini götürərdik, çünki onu öldürmək bu məlumatı itirməyimizə səbəb olardı. Ancaq yaddaş zibilini götürmək burada əhatə dairəsindən kənardadır. Biz bunu artıq etdiyimizi güman edəcəyik və xitam verməyə davam edəcəyik.

Bu prosesi öldürmək üçün kill əmrindən istifadə edə bilərik. Lakin proses kök kimi işlədiyi üçün bu prosesi öldürmək üçün imtiyazları artırmaq üçün sudo istifadə etmək yaxşı fikirdir. Gəlin prosesi öldürməyə çalışaq. Nəzərə alın ki, 2062 PID ilə əvəz etməli olacaqsınız. > komandanın çıxışı.top


ubuntu@tryhackme:~$ sudo kill 633

Burada biz prosesin PID-ni öldürmə əmrinə parametr kimi verdik. Çıxış olaraq heç bir səhv almırıq, ona görə də prosesin uğurla başa çatdığına inanırıq. Üst komanda ilə yenidən yoxlayaq.
Tasks: 187 total,   2 running, 183 sleeping,   0 stopped,   2 zombie
%Cpu(s): 34.6 us,  3.8 sy,  0.0 ni, 53.8 id,  0.0 wa,  0.0 hi,  0.0 si,  7.7 st
MiB Mem :   3933.8 total,   2094.9 free,    632.6 used,   1206.2 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   2983.9 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND   
   2257 root      20   0    2488   1424   1332 R  93.8   0.0   1:59.16 a         
      1 root      20   0  104360  12052   8596 S   0.0   0.3   0:04.53 systemd   
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd  
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp    
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_+ 
      5 root      20   0       0      0      0 I   0.0   0.0   0:00.56 kworker/+ 
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/+ 
      9 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 mm_percp+ 
     10 root      20   0       0      0      0 S   0.0   0.0   0:00.12 ksoftirq+ 
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.63 rcu_sched 
     12 root      rt   0       0      0      0 S   0.0   0.0   0:00.01 migratio+ 
     13 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/0   
     14 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/1   
     15 root      rt   0       0      0      0 S   0.0   0.0   0:00.32 migratio+ 
     16 root      20   0       0      0      0 S   0.0   0.0   0:00.14 ksoftirq+ 
     18 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/+ 
     19 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kdevtmpfs 
     20 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 netns     
     21 root      20   0       0      0      0 S   0.0   0.0   0:00.00 rcu_task+ 
     22 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kauditd   
     23 root      20   0       0      0      0 S   0.0   0.0   0:00.00 xenbus    
     24 root      20   0       0      0      0 S   0.0   0.0   0:00.03 xenwatch  
     25 root      20   0       0      0      0 S   0.0   0.0   0:00.00 khungtas+ 
     26 root      20   0       0      0      0 S   0.0   0.0   0:00.00 oom_reap+ 
     27 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 writeback 
     28 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kcompact+ 
     29 root      25   5       0      0      0 S   0.0   0.0   0:00.00 ksmd      
     30 root      39  19       0      0      0 S   0.0   0.0   0:00.00 khugepag+ 

Vay! Proses hələ də davam edir. Əmrimiz işləmədi, yoxsa nə? Gözləyin, PID və ZAMAN da dəyişdi. Deyəsən, biz prosesi uğurla öldürdük, amma birtəhər yenidən dirildi. Nə baş verdiyini görək.


Checking the Cronjobs
---------------------

Proseslə baş verənlərə dair ilk ipucumuz cronjobs-da olacaq. Cronjobs kompüterdən müəyyən bir intervalla bizim adımıza yerinə yetirməsini xahiş etdiyimiz tapşırıqlardır. Tez-tez burada avtomatik başlanğıc proseslərin izlərini tapa bilərik.



Cronjobları yoxlamaq üçün crontab -l əmrini icra edə bilərik. Aşağıdakı terminalda cronjob formatını anlamağa kömək edə biləcək şərhlərdə (# simvolu ilə başlayan sətirlər) gözəl təsvir göstərilir, ardınca hazırda aktiv olan cronjoblar (# simvolu olmadan başlayan sətirlər). 



ubuntu@tryhackme:~$ crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
@reboot sudo runuser -l ubuntu -c 'vncserver :1 -depth 24 -geometry 1900x1200'
@reboot sudo python3 -m websockify 80 localhost:5901 -D

Deyəsən, prosesimizi burada tapmaq bizim bəxtimiz gətirmir. Biz görürük ki, istifadəçi tərəfindən idarə olunan yeganə cronjoblar VNC serverini idarə etməkdir.

Ancaq gözləyin, proses kök kimi işləyirdi və hər istifadəçinin öz cronjobları var, bəs niyə kök istifadəçi kimi cronjobları yoxlamırıq? Gəlin istifadəçini kökə keçirək və orada nəsə tapıb tapmayacağımızı görək. Biz əvvəlcə sudo su istifadə edərək istifadəçini dəyişdiririk ki, bu da istifadəçimizi root-a keçir. Sonra cronjobları yenidən yoxlayırıq.

ubuntu@tryhackme:~$ sudo su
root@tryhackme:/home/ubuntu# crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command


Yaxşı, çətin şans! Burada cronjob da işləmir. Başqa nə ola bilər?



Check for Running Services
--------------------------

Ola bilsin ki, prosesi geri qaytara biləcək işləyən xidmətləri yoxlamalıyıq. Lakin prosesin adı olduqca ümumidir və yaxşı ipucu vermir. Biz burada samandan yapışmış ola bilərik, amma sistemdə hansı xidmətlərin işlədiyini görək. 


Bunu etmək üçün, biz systemctl list-unit-files sadalanmas > > > > > > > > > > istifade edirik. Axtardığımız xidmət prosesi bərpa etmək üçün aktivləşdirilməli olduğundan, biz grep-dən bizə yalnız aktiv edilmiş xidmətləri təqdim etmək üçün istifadə edirik.

root@tryhackme:/home/ubuntu# systemctl list-unit-files | grep enabled
proc-sys-fs-binfmt_misc.automount              static          enabled      
-.mount                                        generated       enabled      
dev-hugepages.mount                            static          enabled      
dev-mqueue.mount                               static          enabled      
proc-sys-fs-binfmt_misc.mount                  disabled        enabled      
snap-amazon\x2dssm\x2dagent-5163.mount         enabled         enabled      
snap-amazon\x2dssm\x2dagent-7628.mount         enabled         enabled      
snap-core-16202.mount                          enabled         enabled      
snap-core18-2790.mount                         enabled         enabled      
snap-core18-2796.mount                         enabled         enabled      
snap-core20-1361.mount                         enabled         enabled      
snap-core20-2015.mount                         enabled         enabled      
snap-lxd-22526.mount                           enabled         enabled      
snap-lxd-24061.mount                           enabled         enabled      
sys-fs-fuse-connections.mount                  static          enabled      
sys-kernel-config.mount                        static          enabled      
sys-kernel-debug.mount                         static          enabled      
sys-kernel-tracing.mount                       static          enabled      
acpid.path                                     enabled         enabled      
apport-autoreport.path                         enabled         enabled      
cups.path                                      enabled         enabled      
systemd-ask-password-console.path              static          enabled      
systemd-ask-password-plymouth.path             static          enabled      
systemd-ask-password-wall.path                 static          enabled      
session-1.scope                                transient       enabled      
session-c1.scope                               transient       enabled      
a-unkillable.service                           enabled         enabled 

a-unkillable.service                           enabled         enabled

root@tryhackme:/home/ubuntu# systemctl status a-unkillable.service 
● a-unkillable.service - Unkillable exe
     Loaded: loaded (/etc/systemd/system/a-unkillable.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-12-23 12:39:36 UTC; 23min ago
   Main PID: 584 (sudo)
      Tasks: 5 (limit: 4710)
     Memory: 3.6M
     CGroup: /system.slice/a-unkillable.service
             ├─ 584 /usr/bin/sudo /etc/systemd/system/a service
             ├─ 610 /etc/systemd/system/a service
             └─2022 unkillable proc

Dec 23 12:39:36 tryhackme systemd[1]: Started Unkillable exe.
Dec 23 12:39:36 tryhackme sudo[584]:     root : TTY=unknown ; PWD=/ ; USER=root ; COMMAND=/etc/systemd/system/a service
Dec 23 12:39:36 tryhackme sudo[584]: pam_unix(sudo:session): session opened for user root by (uid=0)
Dec 23 12:39:36 tryhackme sudo[644]: Merry Christmas
Dec 23 12:50:48 tryhackme sudo[2010]: Merry Christmas
Dec 23 12:52:10 tryhackme sudo[2026]: Merry Christmas



Getting Rid of the Service
--------------------------

Beləliklə, indi xidməti müəyyən etdikdən sonra ondan xilas olmaq üçün səyahətə çıxaq. İlk addım xidməti dayandırmaq olacaq. 



Bunun üçün kök imtiyazlarına ehtiyacımız ola bilər, ona görə də kök istifadəçiyə keçməli olacağıq.


root@tryhackme:/home/ubuntu# systemctl stop a-unkillable.service 
root@tryhackme:/home/ubuntu# systemctl status a-unkillable.service 
● a-unkillable.service - Unkillable exe
     Loaded: loaded (/etc/systemd/system/a-unkillable.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Sat 2023-12-23 13:04:25 UTC; 20s ago
    Process: 584 ExecStart=/usr/bin/sudo /etc/systemd/system/a service (code=killed, signal=TERM)
   Main PID: 584 (code=killed, signal=TERM)

Dec 23 12:39:36 tryhackme systemd[1]: Started Unkillable exe.
Dec 23 12:39:36 tryhackme sudo[584]:     root : TTY=unknown ; PWD=/ ; USER=root ; COMMAND=/etc/systemd/system/a service
Dec 23 12:39:36 tryhackme sudo[584]: pam_unix(sudo:session): session opened for user root by (uid=0)
Dec 23 12:39:36 tryhackme sudo[644]: Merry Christmas
Dec 23 12:50:48 tryhackme sudo[2010]: Merry Christmas
Dec 23 12:52:10 tryhackme sudo[2026]: Merry Christmas
Dec 23 13:04:25 tryhackme systemd[1]: Stopping Unkillable exe...
Dec 23 13:04:25 tryhackme sudo[584]: pam_unix(sudo:session): session closed for user root
Dec 23 13:04:25 tryhackme systemd[1]: a-unkillable.service: Succeeded.
Dec 23 13:04:25 tryhackme systemd[1]: Stopped Unkillable exe.
root@tryhackme:/home/ubuntu# top

%Cpu(s):  0.8 us,  0.2 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   3933.8 total,   2341.0 free,    751.6 used,    841.2 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   2910.2 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                                                                                     
    926 ubuntu    20   0  380840 159548  64336 S   1.3   4.0   0:19.76 Xtigervnc                                                                                                                                   
   1848 ubuntu    20   0  549424  51348  39156 S   0.7   1.3   0:03.48 mate-terminal                                                                                                                               
   1838 root      20   0  122568  26852   7860 S   0.3   0.7   0:02.13 python3                                                                                                                                     
      1 root      20   0  104464  12112   8544 S   0.0   0.3   0:08.67 systemd                                                                                                                                     
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd                                                                                                                                    
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp                                                                                                                                      
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp                                                                                                                                  
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/0:0H-kblockd                                                                                                                        
      8 root      20   0       0      0      0 I   0.0   0.0   0:00.04 kworker/u30:0-events_power_efficient                                                                                                        
      9 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 mm_percpu_wq                                                                                                                                
     10 root      20   0       0      0      0 S   0.0   0.0   0:00.14 ksoftirqd/0                                                                                                                                 
     11 root      20   0       0      0      0 I   0.0   0.0   0:00.35 rcu_sched                                                                                                                                   
     12 root      rt   0       0      0      0 S   0.0   0.0   0:00.00 migration/0                                                                                                                                 
     13 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/0                                                                                                                                     
     14 root      20   0       0      0      0 S   0.0   0.0   0:00.00 cpuhp/1                                                                                                                                     
     15 root      rt   0       0      0      0 S   0.0   0.0   0:00.29 migration/1                                                                                                                                 
     16 root      20   0       0      0      0 S   0.0   0.0   0:00.11 ksoftirqd/1                                                                                                                                 
     18 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/1:0H-kblockd                                                                                                                        
     19 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kdevtmpfs                                                                                                                                   
     20 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 netns                                                                                                                                       
     21 root      20   0       0      0      0 S   0.0   0.0   0:00.00 rcu_tasks_kthre                                                                                                                             
     22 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kauditd                                                                                                                                     
     23 root      20   0       0      0      0 S   0.0   0.0   0:00.00 xenbus                                                                                                                                      
     24 root      20   0       0      0      0 S   0.0   0.0   0:00.03 xenwatch                                                                                                                                    
     25 root      20   0       0      0      0 S   0.0   0.0   0:00.00 khungtaskd                                                                                                                                  
     26 root      20   0       0      0      0 S   0.0   0.0   0:00.00 oom_reaper                                                                                                                                  
     27 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 writeback                                                                                                                                   
     28 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kcompactd0                                                                                                                                  
     29 root      25   5       0      0      0 S   0.0   0.0   0:00.00 ksmd                                                                                                                                        
     30 root      39  19       0      0      0 S   0.0   0.0   0:00.00 khugepaged                                                                                                                                  
     76 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kintegrityd                                                                                                                                 
     77 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kblockd                                                                                                                                     
     78 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 blkcg_punt_bio                                                                                                                              
     80 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 tpm_dev_wq                                                                                                                                  
     81 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 ata_sff                                                                                                                                     
     82 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 md                                                                                                                                          
     83 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 edac-poller                                                                                                                                 
     84 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 devfreq_wq                                                                                                                                  
     85 root      rt   0       0      0      0 S   0.0   0.0   0:00.00 watchdogd  

root@tryhackme:/home/ubuntu# systemctl disable a-unkillable.service 
Removed /etc/systemd/system/multi-user.target.wants/a-unkillable.service.

Yaxşı, statusun hələ də yükləndiyini görə bilərik, lakin deaktivdir. Problem ondadır ki, xidmət hələ də sistemdə mövcuddur. Xidməti tamamilə silmək üçün faylları fayl sistemindən də silməli olacağıq. Gəlin bunu edək. Burada xidmətin yerinin /etc/systemd/system/[redacted] və prosesin yerinin isə /etc/systemd/system/a olduğunu görürük. Xidməti həmişəlik ləğv etmək üçün gəlin bu iki faylı silək.

root@tryhackme:/home/ubuntu# rm -rf /etc/systemd/system/a
root@tryhackme:/home/ubuntu# rm -rf /etc/systemd/system/a-unkillable.service
root@tryhackme:/home/ubuntu# systemctl status a-unkillable.service
Unit a-unkillable.service could not be found.


What is the name of the service that respawns the process after killing it?
Answer: a-unkillable.service

What is the path from where the process and service were running?
Answer: /etc/systemd/system/


The malware prints a taunting message. When is the message shown? Choose from the options below.

1. Randomly

2. After a set interval

3. On process termination

4. None of the above
Answer: 4

