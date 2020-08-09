# Building django Hash tag Tutorial

## Django 해시태그 구현 - 라이브러리 사용하기

### 사용한 라이브러리

- django-taggit
- django에서 사용하는 해시태그 구현 모듈이 몇 개 있는데, 그 중에 가장 많이 사용하는 형태의 모듈이다.
- Node.js 구현할 땐 `M:N` 데이터베이스 구조를 사용해서 다대다 관계를 작업했는데, django는 좀 더 사용이 편안했다.
- 수강생 코칭을 목적으로 자료를 찾았는데, 오히려 내가 배운 격이 됐다.

### 기록해볼만한 파트

1. models.py

```python
    from django.db import models
    from taggit.managers import TaggableManager #settings.py에서 'taggit' 추가 이후 사용가능


    class Post(models.Model):
        title = models.CharField(max_length=250)
        description = models.TextField()
        published = models.DateField(auto_now_add=True)
        slug = models.SlugField(unique=True, max_length=100) # slugField를 사용해서 키워드를 url로 전달해주는 작업을 모델에서 진행이 반드시 필요하다.
        tags = TaggableManager() # 받아오는 해시태그 값.

        def __str__(self):
            return self.title
```

2. views.py

```python
from django.shortcuts import render, get_object_or_404 # 이곳 세팅은 장고 써본 사람이면 아는 파트니까 패스

from .models import Post
from .forms import PostForm

from taggit.models import Tag #해시태그 모델 받아오기
from django.template.defaultfilters import slugify #태그 값을 중심으로 게시글 필터링하기


def home_view(request):
    posts = Post.objects.all()
    common_tags = Post.tags.most_common()[:4]
    form = PostForm(request.POST)
    if form.is_valid():
        newpost = form.save(commit=False)
        newpost.slug = slugify(newpost.title) # 태그가 걸린 게시글을 필터링할 때 글 제목을 url로 보내서 필터링해준다. slugify 사용법은 django 공식 문서에 자세하게 나와있다.
        newpost.save()
        form.save_m2m() #반드시 이 작업이 해시 태그 구현을 위해 필요하다.
    context = {
        'posts': posts,
        'common_tags': common_tags,
        'form': form,
    }
    return render(request, 'home.html', context)


def detail_view(request, slug):
    post = get_object_or_404(Post, slug=slug)
    context = {
        'post': post,
    }
    return render(request, 'detail.html', context)


def tagged(request, slug):
    tag = get_object_or_404(Tag, slug=slug)
    common_tags = Post.tags.most_common()[:4]
    posts = Post.objects.filter(tags=tag)
    context = {
        'tag': tag,
        'common_tags': common_tags,
        'posts': posts,
    }
    return render(request, 'home.html', context)

```

- 위의 2개가 해시태그 구현하는 것의 핵심이고, 작업을 이해하는 과정은 전혀 어렵지 않았다.
- 그리고 html에서 작업할 때,

```html
<div style="display: flex;">
  {% for tag in post.tags.all %}
  <a href="{% url 'tagged' tag.slug %}" class="mr-1 badge badge-info"
    >#{{ tag }}</a
  >
  <!-- 태그를 가져올 때 tagged라는 urls.py에서 태그에 걸린 slug를 찾아간다는 점이 사용하면서 신선했다. -->
  {% endfor %}
</div>
```

- 해시태그는 input 값 내에서 `띄어쓰기`, `,`, 등을 사용해서 작업이 가능하다.

### 더 파볼 부분

- 해시태그 로직을 정확하게 이해해서 `M:N 하드코딩` 해보기

- 해시태그 값을 `Google Map API`에 전달해서 해시태그 값들을 중심으로 장소 필터링해보기

- Instagram, Facebook 등 말고 사용할 수 있는 방법이 뭐가 있을까?

- Client의 행동은 언제나 서비스 내에서 다양하게 진행되기 때문에 다대다 형태의 매핑은 중요성이 매우 높다.
