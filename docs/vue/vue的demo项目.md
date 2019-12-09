### 一、使用Vue_CLI_3创建项目

1. 先升级vue_cli

   ```
   cnpm uninstall -g vue-cli
   cnpm install -g @vue/cli
   ```

   注意：需要用管理员模式打开命令窗口运行，输入cmd->右键管理员运行

2. 初始化项目

   ```
   vue create vicente-demo-vue
   ```

   

3. 选择自定义的功能Manually select features，空格选择

   ```
   Vue CLI v3.7.0
   ? Please pick a preset: Manually select features
   ? Check the features needed for your project:
    (*) Babel
    ( ) TypeScript
    ( ) Progressive Web App (PWA) Support
    (*) Router
    (*) Vuex
    (*) CSS Pre-processors
    (*) Linter / Formatter
   >(*) Unit Testing
    ( ) E2E Testing
   ```

4. 总体配置

   ```
   Vue CLI v3.7.0
   ? Please pick a preset: Manually select features
   ? Check the features needed for your project: Babel, Router, Vuex, CSS Pre-processors, Linter, Unit
   ? Use history mode for router? (Requires proper server setup for index fallback in production) Yes
   ? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): Less
   ? Pick a linter / formatter config: Prettier
   ? Pick additional lint features: Lint on save, Lint and fix on commit
   ? Pick a unit testing solution: Jest
   ? Where do you prefer placing config for Babel, PostCSS, ESLint, etc.? In dedicated config files
   ? Save this as a preset for future projects? Yes
   ? Save preset as: vicente-vue
   ```

5. 安装ant-design-vue插件，同时安装多个插件使用空格

   ```
   npm i ant-design-vue moment
   ```

6. 放到github上

   ```
   git remote add origin https://github.com/visionsws/vicente-vue.git
   git push -u origin master
   ```

   

### 二、开发

1. 运行项目

   ```
   npm install
   npm run serve
   ```

2. 引入ant-design

   ```
   //在main.js中添加上
   import Antd from "ant-design-vue";
   import 'ant-design-vue/dist/antd.less';
   
   Vue.use(Antd);
   ```

3. 配置less的处理，在项目的 (和 `package.json` 同级的) 根目录中创建`vue.config.js` 可选的配置文件，它会被 `@vue/cli-service` 自动加载，然后在文件上添加配置

   ```
   // vue.config.js
   module.exports = {
       css: {
           loaderOptions: {
               less: {
                   // 这里的选项会传递给 less-loader
                   javascriptEnabled: true
               },
           }
       }
   }
   
   ```

4. 配置路由

   ```
    {
         path: "/user",
         component: () =>
           import(/* webpackChunkName: "about" */ "./layouts/UserLayout"),
         children: [
           {
             path: "/user",
             redirect: "/user/login"
           },
           {
             path: "/user/login",
             name: "login",
             component: () =>
               import(/* webpackChunkName: "user" */ "./views/user/Login")
           },
           {
             path: "/user/register",
             name: "register",
             component: () =>
               import(/* webpackChunkName: "user" */ "./views/user/Register")
           }
         ]
       },
   ```

   在UserLayout中药添加上<router-view></router-view>标签

5. 配置布局

   





