## 更进一步

准备深入学习？以下是一些精选资源，涵盖 Swift 的各种功能。

<ul class="go-further-list">
  {% for resource in site.data.go_further %}
  <li class="resource{% if resource.featured %} featured{% endif %}">
      {% if resource.thumbnail_url %}
        <img class="thumbnail" src="{{ resource.thumbnail_url }}"/>
      {% elsif resource.content_type == "article" %}
        <img class="thumbnail" src="/assets/images/getting-started/article-thumbnail.jpg"/>
      {% endif %}

      <h3>{{ resource.title }}</h3>
      <div class="description">
        {{ resource.description | markdownify }}
      </div>
      
      <a href="{{ resource.content_url }}" class="cta-secondary{% if resource.external %} external" target="_blank"{% else %}"{% endif %}>
        {% if resource.content_type == "video" %}
        观看视频
        {% elsif resource.content_type == "article" %}
        阅读文档
        {% elsif resource.content_type == "book" %}
        阅读手册
        {% else %}
        查看资源
        {% endif %}
      </a>
  </li>
  {% endfor %}
</ul>

想要了解更多信息？在 [文档](/documentation/) 中，您可以找到与 Swift 项目相关的资源、参考资料和指南，包括 [API 设计指南](/documentation/api-design-guidelines/)。
