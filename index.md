---
layout: page
tagline: 点滴思考
---
{% assign abstract_list = site.posts%}
{% include AH/abstract_list %}


<ul class="tag_box inline">
  {% assign categories_list = site.categories %}
  {% include JB/categories_list %}
</ul>


{% for category in site.categories %} 
  <h2 id="{{ category[0] }}-ref">{{ category[0] | join: "/" }}</h2>
  <ul>
    {% assign pages_list = category[1] %}  
    {% include JB/pages_list %}
  </ul>
{% endfor %}


<ul class="tag_box inline">
  {% assign tags_list = site.tags %}  
  {% include JB/tags_list %}
</ul>


{% for tag in site.tags %} 
  <h2 id="{{ tag[0] }}-ref">{{ tag[0] }}</h2>
  <ul>
    {% assign pages_list = tag[1] %}  
    {% include JB/pages_list %}
  </ul>
{% endfor %}



<h2>All Pages</h2>
<ul>
{% assign pages_list = site.pages %}
{% include JB/pages_list %}
</ul>


<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

<pre class="prettyprint linenums">
<code>#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;unistd.h&gt;

int main(int argc, char **argv)
{
  int result;

  opterr = 0;  //使getopt不行stderr输出错误信息

  while( (result = getopt(argc, argv, "ab:c::")) != -1 )
  {
    switch(result)
    {
      case 'a':
           printf("option=a, optopt=%c, optarg=%s\n", optopt, optarg);
           break;
        case 'b':
           printf("option=b, optopt=%c, optarg=%s\n", optopt, optarg);
           break;
        case 'c':
           printf("option=c, optopt=%c, optarg=%s\n", optopt, optarg);
           break;
        case '?':
          printf("result=?, optopt=%c, optarg=%s\n", optopt, optarg);
          break;
        default:
           printf("default, result=%c\n",result);
           break;
       }
    printf("argv[%d]=%s\n", optind, argv[optind]);
  }
  printf("result=-1, optind=%d\n", optind);   //看看最后optind的位置

  for(result = optind; result &lt; argc; result++)
     printf("-----argv[%d]=%s\n", result, argv[result]);

 //看看最后的命令行参数，看顺序是否改变了哈。
  for(result = 1; result &lt; argc; result++)
      printf("\nat the end-----argv[%d]=%s\n", result, argv[result]);
  return 0;
}
</code></pre>