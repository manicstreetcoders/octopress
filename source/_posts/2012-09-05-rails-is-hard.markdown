---
layout: post
title: "Rails Is Hard"
date: 2012-09-05 10:55
comments: true
categories: 
---

코딩에서 손을 놓고 사업과 매니징만 한지 8년이 지났었다. 
내 손으로 할 수 있는 일이 점점 없어지자 갑갑해졌다.
머리속에 떠오르는 아이디어들은 어디서인가 읽은 듯한 느낌이 드는 신선하지 
않은 것들.
새로운 아이디어를 가지기 위해서는, 
Hands-on Experience 가 필요하다는 생각이 들었다.
LEGO 블럭을 가지고 꾸준히 놀아봐야, 손이 가는대로 뭔가를 만들 수 있지 않을까? 
블럭으로 자동차를 만들다가 소방차를 만들고, 비행기를 만들다가 공항을 만드는 것같은 느낌으로.

이 정도의 생각으로 몇 개월동안 아래의 기술들을 사용해서 뭔가를 만들어보려고 노력했다.

그리고 이제는 프로토타입 정도는 혼자 만들 수 있게 되었다. 좀 욕심내면 Parse.com 의 프로토타입 정도는 혼자 만들 수 있겠다 싶다. :-)

아 그런데, 요즘의 웹 개발은 너무 복잡하다.
물론 대부분의 기술이 Open Source 인 덕분에, 커뮤니티도 활성화되어 있고,
참고할만한 리소스들도 많이 있다. 
클라우드 서비스들이 많아서 돈도 얼마 들지 않는다.
그 덕택에 감각이 있는 개발자라면 그럭저럭 쉽게 배워 뭔가를 만들어 낼 수 있기는
하겠지만, 마스터하기는 쉽지 않다. 배워야 할 것들이 너무 많고, 새로운 것들이 계속해서 쏟아져 나오고 있다. 책들이 소프트웨어의 버전업을 따라가지 못하는 상황이다.

이러쿵저러쿵했지만, 그래도 다양한 기술들이 각자 발전해나가면서 멋진 세상이 만들어지고 있다.

예를 들면, 지금 이 시간에도 아마존 EC2 (도쿄) 와 Heroku (버지니아) 와 
Linode (도쿄) 의 데이터센터에서 내가 만든 뭔가가 돌아가고 있다. 
유지하는데 큰 비용이 필요치 않다. 
10 년전에는 생각도 할 수 없었던 일이 벌어지고 있다. 
(그리고 한국에는 실체가 아무것도 없다는 점이 웬지 근사하다. 
케이맨 군도에 설립한 페이퍼 컴패니같은 느낌이랄까)
클라우드와 모바일, 트위터가 이런 변화를 만들어내고 있다고 생각한다.

다시 주제로 돌아와서, 
[A blog in 15 minutes with Ruby on Rails](http://vimeo.com/5362441) 
에서는 Rails 를 쓰면 블로그를 15 분만에 개발할 수 있다고 이야기한다.
하지만 Rails 를 제대로 쓸려면, 
"Rails 가 어떻게 대신 해주는지, Rails 가 미리 가정하고 해버리는 것들은 무엇인지" 를 알아야 한다. 이러다가 HTTP 프로토콜보다 훨씬 더 복잡한 Routing 이나 Controller, Rack 인터페이스 소스를 뒤져야 하는 상황까지 가면, 배보다 배꼽이 더 커진 건 아닌지 자문하게 된다.

또한 RoR 은 많은 dependencies 를 가지고 있다. [Ruby on Rails Has Many Dependencies](http://www.readysetrails.com/index.php/181/this-is-why-learning-rails-is-hard/)

마지막으로 Yehuda Katz 의 강의 [Why Rails is Hard (Railsberry 2012)](http://youtu.be/2Ex8EEv-WPs)

<br \>
<h2>하다보니까 익혀야 할 기술들 - Whoa!</h2>
<h3>Basic Web Concepts</h3>
* HTTP
* DOM
* HTML5
* CSS3

<h3>Languages, Mark-up Languages & Data Exchange Formats</h3>
* Ruby
* JavaScript
* CoffeeScript
* Sass
* Less
* ERB
* YAML
* HAML
* Markdown
* JSON
* Jade (Node.js template engine)

<h3>Databases, Web Servers, Caches & Proxies</h3>
* Unicorn
* [Thin](http://code.macournoyer.com/thin/)
* Nginx
* PostgreSQL
* Londiste for master/slave replication.
* [repmgr](https://github.com/greg2ndQuadrant/repmgr)
* MySQL
* Redis
* Memcached
* MemcacheDB
* HAproxy
* Varnish
* statsd
* Dogslow
* Solr for searching

<h3>Performance Monitoring Tools</h3>
* Monit
* New Relic
* Site24x7
* Pingdom

<h3>Deployments & Configuration Managements</h3>
* Capistrano
* Rubber
* Puppet
* Chef
* Asgard by NetFlix
* Rubber for EC2

<h3>OS Administrations</h3>
* Ubuntu
* OS X

<h3>Installers & Managers</h3>
* Homebrew
* rbenv
* rvm
* Bundler
* [RubyGems](http://rubygems.org)
* [npm](https://npmjs.org)
* [Twitter Bower](https://github.com/twitter/bower)

<h3>Cloud Platforms, VPS, PaaS, SaaS</h3>
* Amazon EC2, S3, Route 53, CloudFront, RDS
* Heroku
* Uhuru AppCloud
* Engine Yard
* Joyent
* Google AppEngine
* Linode (VPS)
* Brightbox
* Parse.com (A backend for iOS & Android Apps)
* Fastly.com (CDN)
* CoudFlare (CDN)
* [cdnjs](http://cdnjs.com) for JS hosting
* [phpfog](http://phpfog.com) for PHP hosting
* [nodejitsu](http://nodejitsu.com) for Node.js hosting
* [SendGrid](http://sendgrid.com) for E-mail delivery
* [Zemanta](https://vimeo.com/46745200) for Social Linking
* [DirectedEdge](http://www.directededge.com) for Recommendation (Amazon-like)
* [PayGate](http://www.paygate.net) for Payment (Korea)
* [Twilio](http://www.twilio.com) for SMS & VOIP

<h3>Tools</h3>
* git
* GitHub
* ssh
* Mercurial SCM
* Project Hosting by Google Code
* Codeplane

<h3>Rails & The Asset Pipeline</h3>
* MVC
* ORM
* TDD 
* REST
* Routing 
* CLI i.e. rake, rails
* REPL i.e. rails console, rails dbconsle, irb
* Sprockets & [Turbo Sprockets](https://github.com/ndbroadbent/turbo-sprockets-rails3)

<h3>Testing Driven Development</h3>
* RSpec
* Cucumber

<h3>JavaScript Frameworks, AMDs, Templates, & Libraries</h3>
* AJAX
* jQuery & jQuery UI
* MooTools
* Backbone.js
* PhantomJS
* Ember.js
* YUI3
* Socket.IO
* RequireJS - a JavaScript file and module loader.
* Underscore.js - a templating library
* Prototype.js
* [Yeoman.io](http://yeoman.io)
* [expressjs](http://expressjs.com)
* [mustache.js](https://github.com/janl/mustache.js/) - a templating library
* CommonJS
* Async - a utility module for working with asynchronous JavaScript.

<h3>JavaScript Tools</h3>
* JSLint
* jsfiddle.net

<h3>CSS Frameworks & CSS Hacks</h3>
* 960Grid.gs
* Normalize.css
* Reset.css
* Modernizr.css
* HTML5 Boilerplate
* Bootstrap
* Zurb Foundation
* [Preboot.less](https://github.com/markdotto/Preboot.less)
* Compass

### Mobile
* popapp.in
* Adobe PhoneGap Build
* Sencha

<h3>Audio/Video</h3>
* [jPlayer](http://www.jplayer.org)
* [MediaElement.js](http://mediaelementjs.com)

<h3>Editor that can be used in web pages</h3>
* [pagedown](http://code.google.com/p/pagedown/)
* [CKEditor](http://ckeditor.com)

<h3>Map API</h3>
* [GMaps.js](https://github.com/HPNeo/gmaps)
* [MapBox](http://mapbox.com)
* OpenStreetMap
* Skyhook

<h3>Comment Add-ons</h3>
* Disqus
* Facebook comments

<h3>Editors, IDEs, Terminals & Screen Multiplexers</h3>
* vim, spf13-vim, [F/ Vim](https://github.com/factorylabs/vimfiles) 
* TextMate & Bundles (i.e. CoffeeScript bundle by jashkenas)
* Sublime Text 2
* Zen Coding
* screen
* tmux
* iTerm2
* WebStorm

<h3>Builders</h3>
* CodeKit
* brunch.io
* [gruntjs](http://gruntjs.com)
* LiveReload([Walkthrough](http://www.youtube.com/watch?feature=player_embedded&v=EZ8vy_cNMVQ))

<h3>Analytics & Conversion Optimizations</h3>
* Google Analytics
* Woopra
* KISSmetric
* MixPanel

<h3>Speed Optimizations</h3>
* yslow
* PageSpeed
* [HTTP Archive(HAR)](http://www.igvita.com/2012/08/28/web-performance-power-tool-http-archive-har/)

<h3>Ad</h3>
* Admob

<h3>CMS, Blog Engines, Static Website Generators, & Web builders</h3>
* XpressEngine
* GnuBoard
* Refinery CMS
* WordPress
* Drupal
* Octopress
* [Monologue](https://github.com/jipiboily/monologue)
* [TextPress](http://textpress.shameerc.com/)
* [wix](www.wix.com)
* [Pantheon](http://www.getpantheon.com) (for Drupal)
* [Cactus](https://github.com/koenbok/Cactus)
* [Jekyll](https://github.com/mojombo/jekyll)

<h3>Crowdsourcing platforms</h3>
* [CloudSpokes](http://www.cloudspokes.com)
* [crowdSPRING](http://www.crowdspring.com)
* [99designs](http://99designs.com)
* [BrandCrowd](http://www.brandcrowd.com)
* [LOGOGALA](http://www.logogala.com)
* [WS LOGOS](http://wslogos.com)
* [logopond](http://logopond.com)
* [DesignNLogo](http://designnlogo.com)
* [HIRE the WORLD](http://www.hiretheworld.com)
* [Sortfolio](http://sortfolio.com)

<h3>Project Management & Issue Tracking</h3>
* Basecamp
* Asana
* HipChat
* [Lighthouse](http://lighthouseapp.com) (Issue tracking)
* [Pivotal Tracker](http://www.pivotaltracker.com/)
* Trello

<h3>Online Learning</h3>
* RailsCasts
* Udacity
* Coursera
* Khan Academy
* Udemy
* MIT OCW
* iTunes-U Stanford
* Vimcasts
* Rails Lab
* Codecademy
* [tuts+](http://tutsplus.com)

<h3>Blogs</h3>
* [Ryan Tomakyo](http://tomayko.com)
* [Ilya Grigorik](http://igvita.com) 
* [Paul Irish](http://paulirish.com) 
* [Pat Shaughnessy](http://patshaughnessy.net)
* [Addy Osmani](http://addyosmani.com/blog)
* [John Dyer](http://johndyer.name)
* [Adam Wiggins](http://adam.heroku.com)
* [Bret Victor](http://worrydream.com)
* [Salvatore Sanfilippo aka antirez](http://antirez.com) 
* [Jamis Buck](http://weblog.jamisbuck.org)
* [Pratik Naik](http://m.onkey.org)
* [Virtuous Code](http://devblog.avdi.org)
* [David Walsh](http://davidwalsh.name/)
* [Robby on Rails](http://www.robbyonrails.com)
* [Mind Mining Medium](http://tech.karbassi.com/)
* [Eigenjoy ](http://eigenjoy.com/) (for Big-data, ruby)
* [Headius](http://blog.headius.com/)

<h3>Conferences</h3>
* [RailsConf 2012](http://confreaks.com/events/railsconf2012)
* [jQuery Conference San Francisco 2012](http://confreaks.com/events/jqcon2012)

### Books
* How Google Tests Software - James Whittaker, Jason Arbon, & Jeff Carollo
* Running Lean, Iterate from Plan A to a Plan That Works - Ash Maurya
* Elemental Design Patterns - Jason M. Smith
* Elements of Programming - Stepanov and Smith
* iOS Programming, The Big Nerd Ranch Guide, 3rd Ed. - Joe Conway & AAron Hillegass
* HTML5 Developer's Cookbook - Chuck Hudson and Tom Leadbetter
* Specification by Example - Gojko Adzic

#### JavaScript
* [Eloquent JavaScript](http://eloquentjavascript.net)
* [JavaScript Garden](http://bonsaiden.github.com/JavaScript-Garden/)
* [JavaScript Patterns]
* [JavaScript The Definitive Guide]
* [Maintainable JavaScript]
* [Principles Of Writing Idiomatic JavaScript]
* [JavaScript: The Good Parts]

#### Rails
* [Ruby on Rails 3 Tutorial]
* [Agile Web Development with Rails]
* [The Rails 3 Way]
* [Rails Recipe]

<h3>Etc</h3>
* Adobe Shadow
* MAMP
* [TestFlight](http://testflightapp.com) (for Distributing iOS Beta App)
* [BuiltWith.com](http://builtwith.com)
* [WhoIsHostingThis?](http://whoishostingthis.com)
