```bash
#!/bin/bash
clear
#dpkg --get-selections | awk -F "mp3info" '{print $1}'
# if ["`dpkg --get-selections | awk 'print $0'`" = ""]
#then 
  sudo apt install mp3info
#fi

echo -ne "\033[35mУкажите директорию (абсолютный путь), информацию о файлах в которой вы хотели бы получить: \033[0m" && read hdir
# home_dir=`pwd`
home_dir="${hdir}"

output='pwd'/output.csv
touch $output
rm $output
exec 1>$output
#exec 2>errors.txt
IFS=$'\n'

#making a header of *.csv
echo "name,type,size,rights,date,duration,dir" 

function myls()
{
i=0 #counter to skip the first line (total:0) in ls -l

#start collecting info about file
for line in `ls -l $1` #--group-directories-first
do
i=$(( $i + 1 ))

 if [ $i -eq 1 ] 
  then 
   continue
 fi
 fil=`echo $line | awk '{print $9}'`
 #print name
 name=`echo "$fil" | awk -F '.' '{print $1}' `  #name is everything before dot
 echo -n "$2$name," #>>$output #print without new line
 
 #say is it a directory or print type of file
 if [ -d "$1/$name" ]
 then 
  echo -n "directory," 
 #elif [ -f "$curr_dir/$fil" ]
 else 
   mtype=`echo "$fil" | awk -F '.' '{print $2}' `  #type is everything after dot   
   echo -n "$mtype,"  #print without new line
fi

 #print size of item
 msize=`echo $line | awk '{print $5}'`
 echo -n "$msize," 

 #print rights
 #echo -n "`stat -c %A $fil`," also can be used
 mrights=`echo $line | awk '{print $1}'`
 echo -n "$mrights, " 

 #print date
 mdate=`echo $line | awk '{print $7 $6  $8}'`
 echo -n "$mdate," 

 #print info about audio
 if [ "$mtype" == "mp3" ] 
   then 
   echo -n "`mp3info -p "%m:%s" $1/$fil`," 
 else 
   echo -n ","  
 fi

 #print current dirrectory to ensure we're still where we want to be
 echo -n "$1,"
 #start new line 
 echo " "

 #go down if current folder have another folder
 if [ -d "$1/$name" ]
 then 
  myls "$1/$fil" "$2-" 
 fi
 done
return 0
}

myls `pwd` ""
exit 0

```
