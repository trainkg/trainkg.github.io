## ubuntu

### keyward

```shell

~ 
��ǰĿ¼
```

### base command

```bash
ls
cd
clear
pwd 
whoami #�鿴��ǰ��½�û�
sudo passwd root #�����û�����
ifconfig 
```

### File command

```bash
mkdir 
rm 
cp 
mv
cat   ����ı�����
touch �����µ��ļ�
touch 1.txt
echo 
echo hello >> 1.txt
```

### Edit command 

```bash
nano  ctro + o, ctrl + X (ubuntu �����ı��༭��)
echo 
cat
redirect: >>(׷��) & > (����)
more
man ls | more ( '|' �ܵ�ָ���ǰһ��������������Ϊ�ڶ������������ )
head
man ls | head -10 (ǰ10��)
tail
man ls | tail -10 (��10��)
```

### Systems Comment

```bash
# �鿴OS�汾
cat /proc/version
sb_release -a
# �������Դ�б�ĸ��¡�
sudo apt-get update 
# ���и��°��İ�װ��
sudo apt-get upgrade
#dist-upgrade������������ϵ�ı仯����Ӱ���ɾ������
sudo apt-get dist-upgrade
# ����
sudo reboot 

# ����
find /usr/local | grep xxx
uname -a
�鿴����ϵͳ��Ϣ����
file xxx.so
�鿴�ļ�����
tar -xvzf
��ѹ�ļ�
gzip
ѹ��,������ԭ�ļ�
gunzip
ԭ��ѹ��
```

###  message & jobs
? mnt Ŀ¼�ǹ���Ŀ¼

```bash
sudo mount  �����趨
sudo umount �������
ln -s /exist_file link_name
jobs
kill %n
ps -af 
ps -af | grep java
cut -c num1-num2
xxx --help
man xxx
help
```



