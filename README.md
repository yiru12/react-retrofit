# react-retrofit
A Retrofit like axios implementation for react native

#Setup package.json
```json
"dependencies": {
    "axios":"^0.19.0",
    "@react-native-community/async-storage":"^1.7.1"
  },
  "devDependencies": {
    "@babel/plugin-proposal-decorators":"^7.7.4"
  },
```

# Setup babel.config.js
```js
  plugins: [
    ["@babel/plugin-proposal-decorators", { "legacy": true }]
  ]
```

# Network Usage
```js
import { Get, Token, Auth, OnRefresh } from 'react-retrofit'
import axios from 'axios'
class API {

  // pass access_token to API refresh_token is optional
  /**
    * @param {string} url
    * @param {AxiosRequestConfig} config
  */
  @AUTH('host/oauth')
  auth(data) {
    //transform data
    return {access_token:data.access_token, refresh_token:data.refresh_token}
  }

  //This will be called when status 401 occurred
  //Please return axios or promise with access_token refresh_token is optional
  @OnRefreshToken()
  refreshToken(refreshToken) {
    return axios({
      method: 'post',
      url: "http://xxx.xxx.com/oauth?grant_type=refresh_token&refresh_token="+refreshToken,
    }).then(result => { return { access_token: result.data.access_token } })
  }

  /**
    * @param {string} url
    * @param {AxiosRequestConfig} config
  */
  @Get('host/me')
  me(info) {
    //transform data
    console.log(info)
    return info
  }

  /**
    * @param {string} url
    * @param {AxiosRequestConfig} config
  */
  @Get('/photo')
  @Token //This will pass access_token to url automatically if @AUTH has been called
  photo(photo) {
    //transform data
    console.log(photo)
    return photo
  }
}

const api = new API()
api.auth()

api.me().then(info => {
    //data you transform
    console.log(info)
})

api.photo().then(photo => {
    //data you transform
    console.log(photo)
})
```

# FlatList Usage
```js
import { List } from 'react-retrofit'
import React, {Text} from 'react'
class App extends React.Component {

    const item = ({ name }) => { return (<Text>{name}</Text>) }

    /**
      * @param {React.Component} component
      * @param {string} url
      * @param {React.Component} listItem
      * @param {RetrofitConfig} config
      */
    @List("Main", "host/endpoint", item)

    //api return like [{"name":"Harry"},{"name":"Billy"}]. Attributes will auto bind to itemView

    render() {
        <this.Main />
    }
}
```

# FlatList With API Usage
```js
import React, {Text} from 'react'
import { List, ListWithAPI, Get } from 'react-retrofit'

class API {

  id = 0

  @Get('http://host/endpoint/')
  fetchData(data) {
    return data
  }

  @Get('http://host/endpoint/{id}')
  fetchMore(data) {
    this.id += 1
    return data
  }
}

const api = new API()

class App extends React.Component {

    const item = ({ name }) => { return (<Text>{name}</Text>) }

    /**
      * @param {React.Component} component
      * @param {function name(data) {}} fetchAPI
      * @param {function name(data) {}} fetchNextAPI
      * @param {React.Component} listItem
      * @param {RetrofitConfig} config
      */
    @ListWithAPI("Main", api.fetchData, api.fetchMore, item)

    //api return like [{"name":"Harry"},{"name":"Billy"}]. Attributes will auto bind to itemView

    render() {
        <this.Main />
    }
}
```