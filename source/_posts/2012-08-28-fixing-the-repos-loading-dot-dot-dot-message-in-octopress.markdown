---
layout: post
title: "Fixing the 'Repos loading...' message in Octopress"
date: 2012-08-28 23:01
comments: true
categories: 
---
If your github.js plugin displays a "Loading repos..." message then it's using an old API. This can be fixed by replacing your github.js file with the one depicted below. 

The pull request may or may not get accepted but here it is: <a href="https://github.com/imathis/octopress/pull/732">#732</a>

{% codeblock lang:js Github.js %}
var github = (function(){
  function render(target, repos){
    var i = 0, fragment = '', t = $(target)[0];

      for(i = 0; i < repos.length; i++) {
        fragment += '<li><a href="'+repos[i].html_url +'">'+repos[i].name+'</a><p>'+repos[i].description+'</p></li>';
      }
    t.innerHTML = fragment;
  }
  return {
    showRepos: function(options){
      $.ajax({
          url: "https://api.github.com/users/"+options.user+"/repos?callback=?"
        , type: 'jsonp'
        , error: function (err) { $(options.target + ' li.loading').addClass('error').text("Error loading feed"); }
        , success: function(data) {
          var repos = [];
           if (!data || !data.data) { return; }
           for (var i = 0; i < data.data.length; i++) {
              if (options.skip_forks && data.data[i].fork) { continue; }
              repos.push(data.data[i]);
            }
          repos.sort(function(a, b) {
            var aDate = new Date(a.pushed_at).valueOf(),
                bDate = new Date(b.pushed_at).valueOf();

            if (aDate === bDate) { return 0; }
            return aDate > bDate ? -1 : 1;
          });

          if (options.count) { repos.splice(options.count); }
          render(options.target, repos);
        }
      });
    }
  };
})();

{% endcodeblock %}
