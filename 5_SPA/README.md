# SPA



## SPA 앱 붙여넣기

- Cloud9에서, **app폴더** 하단에 **public**이라는 이름을 가진 폴더를 생성해주세요.
- public폴더 오른쪽 클릭, **New File** 선택
- 새로운 파일명을 **index.html**로 설정합니다.
- **index.html**을 열고 다음과 같이 코드를 붙여넣어주세요.  



```js
<!doctype html>
<html>
  <head>
    <title>UNITHON-AUSG</title>
    <meta charset="utf-8" />
  </head>
  <body>
    <div id="app">
      <h1>AUSG Instagram</h1>
      <div class="upload">
        <input id="f" type="file" accept="image/png, image/gif, image/jpeg, image/bmp, image/x-icon"/>
        <input id="t" type="text" v-model="title" placeholder="title"/>
        <input id="d"type="text" v-model="description" placeholder="description"/>
        <button @click="upload">업로드</button>
      </div>
      <div class="image-posts">
        <div
          class="image-posts__post"
          v-for="imagePost in imagePosts"
          :key="imagePost.id"
        >
          <img :src="imagePost.url" style="width: 100%"/>
          <h2>{{ imagePost.title }}</h2>
          <p>{{ imagePost.description }}</p>
        </div>
      </div>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.16/vue.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.18.0/axios.min.js"></script>
    <script>
      function s3UploadFile(preSignedURL, file) {
        return axios.put(preSignedURL, file, { headers: {
          'Content-Type': 'multipart/form-data',
        } })
      }
      const app = new Vue({
        el: '#app',
        data: {
          file: '',
          title: '',
          description: '',
          imagePosts: [],
        },
        computed: {},
        methods: {
          upload() {
            let url = null
            axios.post(`/generatePresignedUrl`, {
              filename: document.getElementById('f').files[0].name
            })
              .then(({ data }) => {
                url = data.url
                return s3UploadFile(data.presignedUrl, document.getElementById('f').files[0])
              })
              .then(({ data }) => {
                return axios.post(`/images`, {
                  title: this.title,
                  description: this.description,
                  url,
                })
              }).then(({ data }) => {
                this.fetchImagePosts()
              })
              .catch((err) => {
                console.log(document.getElementById('f').files[0])
                console.log(err)
              })
          },
          fetchImagePosts() {
            axios.get(`/images`)
              .then(({ data }) => {
                this.imagePosts = data
              })
          }
        },
        mounted() {
          this.fetchImagePosts()
        }
      })
    </script>
  </body>
</html>
```



## Express 서버에 정적 호스팅하기

아래의 코드를 index.js에 추가해주세요

```js
const express = require('express')
const bodyParser = require('body-parser')
const cors = require('cors')
const app = express()

const Sequelize = require('sequelize')
const sequelize = new Sequelize('DB이름', '마스터사용자이름', '마스터암호', {
    host: '확인한엔드포인트주소',
    dialect: 'mysql',
});

const ImagePost = sequelize.define('image_post', {
    id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
    },
    username: Sequelize.STRING,
    title: Sequelize.STRING,
    description: Sequelize.TEXT,
    url: Sequelize.STRING,
})

ImagePost.sync({ alter: true })

app.use(cors())
app.use(bodyParser.json())

app.get('/', function (req, res) {
    res.json({
        message: "Hello, Like-Lion! We're AUSG!",
    })
})

app.get('/images', function (req, res) {
    ImagePost.findAll()
        .then(function (result) {
            res.json(result)
        })
        .catch(function (error) {
            res.json(error)
        })
})

app.post('/images', function (req, res) {
    ImagePost.create({
        title: req.body.title,
        description: req.body.description,
        url: req.body.url,
    }).then(function () {
        res.json({
            message: 'success!',
        })
    }).catch(function (error) {
        res.json({
            message: 'failed!',
        })
        console.log(error)
    })
})  

const AWS = require('aws-sdk')
const s3 = new AWS.S3({
    region: 'ap-southeast-1',
    signatureVersion: 'v4',
})

app.post('/generatePresignedUrl', function (req, res) {
    const key = `${Date.now()}_${req.body.filename}`
    const params = {
        Bucket: '버킷 이름',
        Key: key,
        ACL: 'public-read',
    }
    const presignedUrl = s3.getSignedUrl('putObject', params)
    res.json({
        key,
        presignedUrl,
    })
})
///-------- 추가
app.use(express.static('public'))  
///-------- 추가

app.listen(3000)
```



- Cloud9 엔드포인트 IP 주소 뒤에  index.html을 붙여서 브라우저에서 실행해보세요. 



오늘 실습은 여기까지입니다.

실습이 끝나셨다면, AWS 서비스 삭제 가이드를 따라 삭제를 진행해주세요. (과금 방지용입니다.)