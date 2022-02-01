---
title: About Me
author: Aritra
permalink: /about/
sidebar:
  nav: nonexistent
---
I am a developer, hobby security researcher, guitarist, anime lover. Try to write about whatever I find interesting.
- 5 Years experience in Software Development
- Current Role: SDE II, Amazon
- Dev Skills: 
    - NodeJS, Express : ★★★★★  
    - React/RN, HTML-CSS-JQ : ★★★★★  
    - Golang : ★★★★  
    - Docker/Kubernetes : ★★★★  
    - CI/CD, Bash, Git : ★★★★  
    - PHP, Python : ★★★  
- Security Skills:
  - Certificate: Offensive Security Certified Professional [ [Verify](https://www.credly.com/badges/f47df179-9b23-4c06-b641-7acf96e82c41/public_url) ]
  - Published CVE: [CVE-2021-23631](https://nvd.nist.gov/vuln/detail/CVE-2021-23631)
- MyAnimeList: [https://myanimelist.net/animelist/legndery](https://myanimelist.net/animelist/legndery)
- Socials:
  {% for social in site.author.links %}<a href="{{social.url}}" rel="nofollow noopener noreferrer"><i class="{{social.icon}} fa-lg" aria-hidden="true"></i></a>{% endfor %}

{% comment %}
{% highlight golang %}
package main 
 
func main() { 
  aritra_chakraborty := Developer { 
    WorkExperience: "5+ years", 
    CurrentRole: "Senior Application Developer, Thoughtworks" 
    Skillz: []map[string]string{ 
      { "NodeJS, Express" : "★★★★★" }, 
      { "React/RN, HTML-CSS-JQ" : "★★★★★" }, 
      { "Golang" : "★★★★" }, 
      { "Docker/Kubernetes" : "★★★★" }, 
      { "CI/CD, Bash, Git" : "★★★★" }, 
      { "PHP, Python" : "★★★" }, 
    }, 
    Projects: []map[string]string{ 
      github: "https://github.com/legndery"
    } 
    Social: []string{ 
      linkedin: "https://www.linkedin.com/in/legndery/",
      StackOverFlow: { "10k+ Rep" : "https://stackoverflow.com/users/1203844/aritra-chakraborty"} 
    } 
  } 
} 
{% endhighlight %}
{% highlight js %}
import DeveloperProfile from './developerProfile'; 
 
class SecurityEnthusiast extends DeveloperProfile { 
  constructor() { 
    this.role = "Security Representative India, Thoughtworks", 
    this.certificates = { 
      "Offensive Security": { 
        "Name": "OSCP", 
        "Link": "https://www.credly.com/badges/f47df179-9b23-4c06-b641-7acf96e82c41/public_url",
      } 
    }, 
    this.achievements = { 
      "Hack The Box": { 
        "rank": "Guru(>90% ownership), Reached Top 100", 
        "profile": "https://www.hackthebox.eu/profile/302236"
      } 
    } 
  } 
} 

const Aritra_Chakraborty = new SecurityEnthusiast(); 
{% endhighlight %}


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
{% endcomment %}