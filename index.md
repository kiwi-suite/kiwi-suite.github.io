---
title: "kiwi Documentation"
keywords: kiwi cmf cms framework
sidebar: none
permalink: index.html
summary: kiwi is a content management framework for developers. 
---

## Available Packages
{% for package in site.data.packages.packages %}
<div class="row" style="margin-top:5px;">
    <div class="col-md-4">
        <a href="https://github.com/{{package}}">{{package}}</a>
    </div>
    <div class="col-md-8">
        <div class="pull-left" style="min-width:120px;">
            <a class="repo-badge" href="https://travis-ci.org/{{package}}" rel="nofollow">
                <img src="https://travis-ci.org/{{package}}.svg?branch=develop" data-canonical-src="https://travis-ci.org/{{package}}" style="max-width:100%;">
            </a>
        </div>
        <div class="pull-left" style="min-width:140px;">
          <a class="repo-badge" href="https://coveralls.io/github/{{package}}" rel="nofollow">
              <img src="https://coveralls.io/repos/github/{{package}}/badge.svg?branch=develop" data-canonical-src="https://coveralls.io/github/{{package}}" style="max-width:100%;">
          </a>      
        </div>
        <div class="pull-left">
            <a class="repo-badge" href="https://packagist.org/packages/{{package}}" rel="nofollow">
                <img src="https://img.shields.io/packagist/v/{{package}}.svg" data-canonical-src="https://packagist.org/packages/{{package}}" style="max-width:100%;">
            </a>
        </div>
        <div class="clearfix"></div>
    </div>
</div>
{% endfor %}
