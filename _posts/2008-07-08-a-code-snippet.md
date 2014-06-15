---
layout: post
title: df
---

{{ page.title }}
===============

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/stat.h>//Cookie: JSESSIONID=1247431066C6C69329F5121CDEC73231\r\n
#include <dirent.h>
#include <unistd.h>

#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>

typedef struct _FileList
{
    char filename[64];
    struct _FileList *next;
}FileList;

//char *host_name = "acm.whu.edu.cn";
int port = 80;

int urlencode(char *in,int inlen,char *out);
void submitcodes(FileList *files,char *hostname,char *srcpath);

char sendbuf[8*1024],altbuf[8*1024];
char httphead[13][256]={
"POST /oak/submit/submit.do HTTP/1.1\r\n",
"Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms-excel, application/vnd.ms-powerpoint, application/msword, application/x-shockwave-flash, */*\r\n",
"Referer: http://acm.whu.edu.cn/oak/submit/submit.jsp\r\n",
"Accept-Language: zh-cn\r\n",
"Content-Type: application/x-www-form-urlencoded\r\n",
"UA-CPU: x86\r\n",
"Accept-Encoding: gzip, deflate\r\n",
"User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.2; SV1; .NET CLR 1.1.4322)\r\n",
"Host: ",
"Content-Length: ",//注意
"Connection: Keep-Alive\r\n",
"Cache-Control: no-cache\r\n",
"\r\n"
};

char userinfo[5][64]={
"user_id=",
"&pass=",
"&problem_id=",
"&language=",
"&source="
};

int main()
{
    //FILE *fp;
    int langid;
    char srcpath[256],id[64],pass[64],host_name[64];
    printf("OJ url : ");
    scanf("%s",host_name);
    printf("user_id : ");
    scanf("%s",id);
    printf("password : ");
    scanf("%s",pass);
    printf("select a language :\n1,gcc.\n2,g++.\n3,java.\n4,pacal.\n");
    scanf("%d",&langid);

    sprintf(userinfo[0]+8,"%s\0",id);
    sprintf(userinfo[1]+6,"%s\0",pass);
    sprintf(userinfo[3]+10,"%d\0",langid-1);
    
    DIR *directory_pointer;
    struct dirent *entry;

    FileList start,*node;
    printf("src file's path : ");
    scanf("%s",srcpath);
    if((directory_pointer=opendir(srcpath))==NULL)
        printf("Error opening %s\n",srcpath);
    else
    {
        start.next=NULL;
        node=&start;
        while((entry=readdir(directory_pointer))!=NULL)
        {
            if(!strcmp(entry->d_name,".")||!strcmp(entry->d_name,".."))
                continue;
            node->next=(FileList *)malloc(sizeof(FileList));
            node=node->next;
            strcpy(node->filename,entry->d_name);
            node->next=NULL;
        }
        closedir(directory_pointer);
        //node=start.next;
    }
    submitcodes(&start,host_name,srcpath);
}

int urlencode(char *in,int inlen,char *out)
{
    int i,len=0;
    for(i=0;i<inlen;i++)
    {
        if(in[i]==' ')
            out[len++]='+';
        else if(!(in[i]>='0'&&in[i]<='9')&&!(in[i]>='a'&&in[i]<='z')&&!(in[i]>='A'&&in[i]<='Z')&&in[i]!='*'&&in[i]!='-'&&in[i]!='.'&&in[i]!='@')
        {
            out[len++]='%';
            sprintf(out+len,"%02X",in[i]);
            len+=2;
        }
        else
            out[len++]=in[i];
    }
    return len;
}

void submitcodes(FileList *files,char *hostname,char *srcpath)
{
    int sd;
        struct sockaddr_in pin;
        struct hostent *nlp_host;

        if((nlp_host=gethostbyname(hostname))==0)
        {
                printf("Error resolving local host\n");
                exit(1);
        }    
        bzero(&pin,sizeof(pin));
        pin.sin_family = AF_INET;
        //pin.sin_addr.s_addr = htonl(INADDR_ANY);
        pin.sin_addr.s_addr = ((struct in_addr *)(nlp_host->h_addr))->s_addr;
        pin.sin_port=htons(port);

        if((sd = socket(AF_INET,SOCK_STREAM,0))==-1)
        {
                printf("Error opening socket\n");
                exit(1);
     }
      
    if(connect(sd,(void*)&pin,sizeof(pin))==-1)
        {
                printf("Error conecting to socket\n");
                exit(1);
        }
    FileList *node=files->next;
        FILE *fp;
        char fullpath[64];

    while(node)
    {
        strcpy(fullpath,srcpath);
        strcat(fullpath,"/");
        strcat(fullpath,node->filename);
        fp=fopen(fullpath,"rb");
        struct stat buf;
        if(stat(fullpath,&buf)!=0)
        {
            printf("file stat fail!\n");
            exit(1);
        }
        char problem_id[64];
        strcpy(problem_id,node->filename);
        int ii=strlen(problem_id)-1;
        while(ii>=0&&problem_id[ii]!='.')ii--;
        if(ii>=0)
            problem_id[ii]=0;

        char *inp,*outp;
        inp=(char *)malloc(buf.st_size);
        outp=(char *)malloc(3*buf.st_size);
        if(buf.st_size!=fread(inp,1,buf.st_size,fp))
        {
            printf("file read fail!\n");
            fclose(fp);
            exit(1);
        }
        fclose(fp);
        int outlen=urlencode(inp,buf.st_size,outp);
        int jj;
        sprintf(userinfo[2]+12,"%s",problem_id);
        strcpy(altbuf,userinfo[0]);//注意
        for(jj=1;jj<5;jj++)
            strcat(altbuf,userinfo[jj]);

        outp[outlen]=0;
        strcat(altbuf,outp);

        int tt = strlen(altbuf);
        sprintf(httphead[8]+6,"%s\r\n\0",hostname);
        sprintf(httphead[9]+16,"%d\r\n\0",tt);
        strcpy(sendbuf,httphead[0]);
        for(jj=1;jj<13;jj++)
            strcat(sendbuf,httphead[jj]);
        strcat(sendbuf,altbuf);
            if(send(sd,sendbuf,strlen(sendbuf),0)==-1)
                {
                    printf("Error in send\n");
                       exit(1);
                }
        node=node->next;
        sleep(1);
       }

       close(sd);
}
