---
layout: post
title: 组合生成代码 
---

{{ page.title }}
===============

<pre>
<code>
public function uni_zuhe($n,$m)//回溯法生成所有组合
{
    if($n<=0)
        return false;
    if($m<0)
        return false;
    $res_arr=array();
    
    $e=array();
    $l=0;
    $e[$l]=1;
    $l=1;
    while($l>0)
    {
        if($l==$m)
        {
            $res_arr[]=$e;
            //print_r($e);
            //echo "</br>";
            if($e[$l-1]+1<=$n)
                $e[$l-1]=$e[$l-1]+1;
            else 
            {
                $l=$l-1;
                if($l>=1)
                    $e[$l-1]=$e[$l-1]+1;
            }
        }
        else 
        {
            if($e[$l-1]+1<=$n)
            {
                $e[$l]=$e[$l-1]+1;
                $l=$l+1;
            }
            else
            {
                $l=$l-1;
                if($l>=1)
                    $e[$l-1]=$e[$l-1]+1;
            }
        }
    }
    //print "n:"+$n+"m:"+$m+"
    //print("</br>count:".count($res_arr));
    //print_r($res_arr);
}
</code>
</pre>
