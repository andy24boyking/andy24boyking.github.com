---
layout: page
tagline: 点滴思考
---
{% include JB/setup %}
## Update Author Attributes

In `_config.yml` remember to specify your own data:
    
    title : My Blog =)
    
    author :
      name : Name Lastname
      email : blah@email.test
      github : username
      twitter : username

The theme should reference these variables whenever needed.
    
## Sample Posts

This blog contains sample posts which help stage pages and blog data.
When you don't need the samples anymore just delete the `_posts/core-samples` folder.

    $ rm -rf _posts/core-samples

Here's a sample "posts list".

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## To-Do

This theme is still unfinished. If you'd like to be added as a contributor, [please fork](http://github.com/plusjade/jekyll-bootstrap)!
We need to clean up the themes, make theme usage guides with theme-specific markup examples.

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