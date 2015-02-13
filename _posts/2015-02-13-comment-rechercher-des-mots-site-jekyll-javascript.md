---
layout: post
title: Recherche dans un site Jekyll statique en JS
date: 2015-02-13 05:14
author: Thomas ASNAR
categories: [jekyll, javascript]
---
J'utilise [Jekyll](jekyllrb.com) pour générer les pages statiques que vous êtes en train de lire.

Le problème d'un site statique, c'est qu'on ne peut pas interroger son serveur web (php, node js ou autre) pour obtenir des informations.

En PHP, par exemple, si l'on veut rajouter un module de recherche sur son site, il suffit d’interroger un contrôleur qui va se charger d'aller chercher dans un modèle les mots recherchés en POST.

Mais qu'à cela ne tienne. Avec un peu de javascript / ajax, c'est facile.

Je ne dis pas que ma solution est la meilleure, mais elle fonctionne. Le principe est de générer un fichier xml avec tous les mots de mon blog et de pouvoir récupérer le titre et l'url du post en question.

J'ai trouvé que le plus simple était d'étoffer un peu le feed.xml proposé par défaut. Après, il est vrai que mon blog n'est pas bien gros, et je me suis permis de mettre tout le contenu dans feed.xml. Mais on peut se restreindre aux mots clés et à un excerpt par exemple.

## [Exemple feed.xml](https://github.com/thomas-asnar/thomas-asnar.github.io/blob/master/feed.xml)

```
--&#45;
layout: null
--&#45;
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ site.title | xml_escape }}</title>
    <description>{{ site.description | xml_escape }}</description>
    <link>{{ site.url }}{{ site.baseurl }}/</link>
    <atom:link href="{{ "/feed.xml" | prepend: site.baseurl | prepend: site.url }}" rel="self" type="application/rss+xml"/>
    <pubDate>{{ site.time | date_to_rfc822 }}</pubDate>
    <lastBuildDate>{{ site.time | date_to_rfc822 }}</lastBuildDate>
    <generator>Jekyll v{{ jekyll.version }}</generator>
    {% for post in site.posts %}
      <item>
        <title>{{ post.title | xml_escape }}</title>
        <description>{{ post.content | xml_escape }}</description>
        <pubDate>{{ post.date | date_to_rfc822 }}</pubDate>
        <link>{{ post.url | prepend: site.baseurl | prepend: site.url }}</link>
        <guid isPermaLink="true">{{ post.url | prepend: site.baseurl | prepend: site.url }}</guid>
        {% for tag in post.tags %}
        <category>{{ tag | xml_escape }}</category>
        {% endfor %}
        {% for cat in post.categories %}
        <category>{{ cat | xml_escape }}</category>
        {% endfor %}
      </item>
    {% endfor %}
  </channel>
</rss>
```

Ensuite, l'idée est de charger avec ajax le fichier feed.xml et de chercher les mots. Puis je mets en forme grâce à javascript (jQuery).

Voici ma fonction. L'avantage de la fonction, c'est que je peux créer ensuite plusieurs events (recherche par menu, par click sur les catégories, etc.)

## Fonction de recherche de mots dans un site statique Jekyll

```
function searchSite(words){
  $.ajax({
    type:'GET',
    url:'/feed.xml',
    dataType: 'xml',
    success:function(xml){
    var heading = $('.page-heading');
    if(heading.length > 0){
      var items = $(xml).find('item:contains('+words+')') ;
      $('.post-list').remove();
      $('.pagination').remove();
      
      
      if(items.length > 0){
        var postListHtml = '<ul class="post-list">';
        items.each(function(index){
          var categoriesButtons = '' ;
          var categories = $(items[index]).find('category') ;
          categories.each(function(indexCat){
            categoriesButtons += '<button type="button" class="btn btn-default btn-xs btn-categories hidden-xs">\
            <span class="glyphicon glyphicon-asterisk"></span>'+$(categories[indexCat]).text()+'</button> ' ;
            ; 
          });

          postListHtml += '\
<li>\
<h3><a class="post-link" href="'+ $(items[index]).find('link').text() +'">'+ $(items[index]).find('title').text() +'</a></h3>\
<footer class="post-footer">'+categoriesButtons+'\
</footer>\
</li>\
          ';
        });
        postListHtml += '</ul>';
        heading.after(postListHtml);
      }else{
        heading.after('<div class="post-list">Aucun Résultat</div>');
      }
    
    
    }else{
      window.location.href = '/#!'+words ;
    }
    
    }
  });
} /* searchSite(words) */
```

## Exemples d'events qui trigger la fonction

```
$(document).on('click','#exec-search',function(){
  searchSite($(this).siblings('input').val());
});
$('#exec-search ~ input').keypress(function (e) {
 var key = e.which;
 if(key == 13)  // the enter key code
  {
    searchSite($(this).val());
  }
});
$(document).on('click','.btn-categories',function(){
  searchSite($(this).text().trim());
});
var slug = window.location.href.split("#!")[1] ;
if(slug){
  searchSite(slug);
}
```