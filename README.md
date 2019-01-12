
2019.01.10 jwxh修改为 中文支持  vue文件管理器  仅仅是修改了部分英文词汇到中文.


如果你要使用这个包,你可以使用npm安装git上的包方法来安装

如果你直接用 npm install 是安装不了这个包的.
同时vuefinder的版本号后面被修改了.

比如版本号 vuefinder2.0.0  被修改了 vuefinder2.0.095 因为相同版本,npm总是去原作者那安装了.

如何安装,请执行下面的代码:
npm install git+https://github.com/jiuwuxiaohun/vuefinder.git --save
或安装了cnpm的情况下
cnpm install git+https://github.com/jiuwuxiaohun/vuefinder.git --save

--------------

# Vuefinder File Manager

#### Readme in progress..

The project is also still in its early stages.

#### List of dependencies:

   - @fortawesome/fontawesome-svg-core,
   - @fortawesome/free-solid-svg-icons,
   - @fortawesome/vue-fontawesome,
   - axios,
   - date-fns/format,
   - dragselect,
   - vue

#### Installation
```
npm install vuefinder
```

#### Usage

````javascript
    import Vue from 'vue';
    import vuefinder from 'vuefinder/src/Finder.vue';

    Vue.component('vuefinder', vuefinder);

    var app = new Vue({
        el: '#app'
    });
````

When you register the sfc component it will be available as simple as the code below.

````vue
<div id="app">
    <vuefinder url="vuefinder" ></vuefinder>
</div>
````

#### Server-Side php library

[vuefinder-php](https://github.com/n1crack/vuefinder-php) //  still in progress..

#### Meanwhile the project screenshot:

![vuefinder ss](ss/1.jpg)

#### and youtube video of first version

[![Vuefinder Demo](https://img.youtube.com/vi/QV0H3NzmQVQ/0.jpg)](https://www.youtube.com/watch?v=QV0H3NzmQVQ)

#### So..
Let me hear what do you think of the project?
