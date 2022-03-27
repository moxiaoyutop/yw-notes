# Linux-小工具脚本

```
#!/bin/bash
#
case $1 in
ns)
if [ -f ns.txt ]; then
rm -rf ns.txt
fi
for i in `cat a`; do
nslookup -query=ns $i | egrep "nameserver = " | head -1 | awk '{print $1,$4}' >> ns.txt
done
echo -e "\033[0;31m 脚本执行完毕,结果输出在ns.txt \033[0m"
;;
cname)
if [ -f cname.txt ]; then
rm -rf cname.txt
fi
for i in `cat a`; do
nslookup -query=cname $i | egrep "canonical name = " | awk '{print $5}' >> cname.txt
done
echo -e "\033[0;31m 脚本执行完毕,结果输出在cname.txt \033[0m"
;;
duibi)
if [ -f xiangton.txt -o -f buton.txt ]; then
rm -rf xiangton.txt buton.txt
fi
for i in `cat a`; do
grep -w $i b > /dev/null
if [ $? -eq 0 ]; then
echo -e $i >> xiangton.txt
else
echo -e $i >> buton.txt
fi
done
echo -e "\033[0;31m 脚本执行完毕,结果输出在xiangton.txt和buton.txt \033[0m"
;;
host)
if [[ -f host.txt ]]; then
rm -rf host.txt
fi
for i in `cat a`; do
host $i | sed ':a;N;s/\n//;ta;' >> host.txt
done
echo -e "\033[0;31m 脚本执行完毕,结果输出在host.txt \033[0m"
;;
curl)
if [ -f ton.txt -o jujue.txt ]; then
rm -rf ton.txt jujue.txt
fi
for i in `cat a`; do
ServiceCode=$(curl -s -m 10 --connect-timeout 10 -l $i -w %{http_code} -I|awk '{print $2}'|awk 'NR==1{print}')
if [ $ServiceCode -eq 301 -o $ServiceCode -eq 200 ]; then
echo -e "服务正常,$i" >> ton.txt
else
echo -e "服务无法访问,$i" >> jujue.txt
fi
done
echo -e "\033[0;31m 脚本执行完毕,结果输出在ton.txt和jujue.txt \033[0m"
;;
ping)
for i in `cat a`
do
{
ping -c 1 $i >> /dev/null
if [ $? -eq 0 ]
then
echo -e $i >> yes.txt
else
echo -e $i >> no.txt
fi
}&
done
echo "\033[0;31m 脚本执行完毕,结果输出在yes.txt和no.txt \033[0m"
;;
esac
```
