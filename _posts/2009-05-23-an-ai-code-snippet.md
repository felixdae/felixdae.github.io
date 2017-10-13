---
layout: post
title: 解决“警察，贼，爸爸，妈妈，儿，儿，女，女”问题的程序
---

{{ page.title }}
===============

<pre>
<code>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

//p, t, f, m, s1, s2, d1, d2
unsigned char mask[8]={128,64,32,16,8,4,2,1};
unsigned char onBoat[16]= {
                            128, 64, 32, 16,//p, t, f, m
                            192, 160, 144, 48,//p&t, p&f, p&m, f&m
                            136, 132, 130, 129,//p&s1, p&s2, p&d1, p&d2
                            40,36,//f&s1, f&s2
                            18,17//m&d1, m&d2
                            };
char arr[16][32]={
                "police",
                "terrorist",
                "father",
                "mother",
                "police and terrorist",
                "police and father",
                "police and mother",
                "father and mother",
                "police and son1",
                "police and son2",
                "police and daughter1",
                "police and daughter2",
                "father and son1",
                "father and son2",
                "mother and daughter1",
                "mother and daughter2"
                };
            
int
translate(int ob)
{
    switch(ob)
    {
        case 128:return 0;
        case 64: return 1;
        case 32: return 2;
        case 16: return 3;
        case 192:return 4;
        case 160:return 5;
        case 144:return 6;
        case 48 :return 7;
        case 136:return 8;
        case 132:return 9;
        case 130:return 10;
        case 129:return 11;
        case 40: return 12;
        case 36: return 13;
        case 18: return 14;
        case 17: return 15;
        default: return -1;
    }
}
int visited1[256], visited2[256], trace[256];

int
isLegal(int state)
{
    if(state>64&&state<128)//!p && t && f||m||s||s||d||d 
        return 0;
    if((state&mask[2])&&(!(state&mask[3]))&&(state&(mask[6]^mask[7])))//f && !m && d||d
        return 0;
    if((!(state&mask[2]))&&(state&mask[3])&&(state&(mask[4]^mask[5])))//m && !f && s||s
        return 0;
    return 1;
}

int
traverse(unsigned char state, int direction, int steps)
{
    int i;
    if(direction)//go
    {
        if(state==0)
        {
            for(i=0; i<steps; i++)
                printf("%s %s\r\n", arr[translate(trace[i])], (i%2? "return":"go"));
            printf("\r\n");
            return 1;
        }
        if(visited1[state]!=0)
            return 1;
        visited1[state]=1;
        for(i=0; i<16; i++)
        {
            if((state&onBoat[i])==onBoat[i])
            {
                if(isLegal(state^onBoat[i])&&isLegal((~state)^onBoat[i]))
                {
                    trace[steps]=onBoat[i];
                    traverse((~state)^onBoat[i], !direction, steps+1);
                }
            }
        }
        visited1[state]=0;
    }
    else//return
    {
        if(state==255)
        {
            for(i=0; i<steps; i++)
                printf("%s %s\r\n", arr[translate(trace[i])], (i%2? "return":"go"));
            printf("\r\n");
            return 1;
        }
        if(visited2[state]!=0)
            return 1;
        visited2[state]=1;
        for(i=0; i<16; i++)
        {
            if((state&onBoat[i])==onBoat[i])
            {
                if(isLegal(state^onBoat[i])&&isLegal((~state)^onBoat[i]))
                {
                    trace[steps]=onBoat[i];
                    traverse((~state)^onBoat[i], !direction, steps+1);
                }
            }
        }
        visited2[state]=0;
    }
    return 1;
}

int
main(int argc, char **argv)
{
    traverse(255, 1, 0);
    return 0;
}
</code>
</pre>
