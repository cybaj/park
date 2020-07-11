<template>
  <div class="wrapper">
  <div class="cover"></div>
  <div class="content">
      <div class="mainContent">
        <div class="element convex" v-on="{ mousedown: doThis, mouseup: doThat, touchstart: doThis, touchend: doThat}">헌화를 하시겠습니까?</div>
        <div class="label">꾹 눌러주세요.</div>
        <div class="portrait">
          <div class="framewrapper">
          <iframe class="portraitframe" :src="portraitsrc"></iframe>
          </div>
        </div>
        <div class="flower">
        <img class="flowerimage" src="https://cdn.pixabay.com/photo/2020/06/08/06/58/chrysanthemum-5273366_1280.jpg" /></div>
        <div class="laying"></div>
      </div>
  </div>
  </div>
</template>

<script>
export default {
  name: 'Park',
  props: {
    msg: String
  },
  data: ()=>{
  return {
    ratio: 0.66,
    offset: 60,
    publicPath: process.env.BASE_URL,
    portraitsrc: '',
    sequence_number: 10,
    done: false,
    working: false,
    current: false
  }},
  mounted: function(){
    const devicewidth = window.screen.width;
    const deviceheight = window.screen.width;
    if (deviceheight >= 650 && devicewidth >= 650) {
      this.portraitsrc = `${this.publicPath}portrait-200.html` 
    } else {
      this.portraitsrc = `${this.publicPath}portrait-100.html`
    }

    console.log(this.portraitsrc)
    console.log(this.publicPath)
  },
  methods: {
    'doThis': function(e) {
      const button = e.target
      button.class = "element concave"
      button.style['animation-duration'] = '4s'
      button.style['animation-name'] = 'down'
      const label = document.getElementsByClassName('label')[0]
      label.style['animation-duration'] = '4s'
      label.style['animation-name'] = 'labeldown'

      this.timeout1 = window.setTimeout(()=>{
        const self = this
        console.log('done')
        if (window.localStorage['isLaid'] != 'true') {
          this.current = true;
          fetch('https://p2pebb68te.execute-api.ap-northeast-2.amazonaws.com/park_counter_stage/number', {
            method: 'POST'
          }).then(function(response) {
              return response.json();
          }).then(function(data) {
              console.log(data);
              self.sequence_number = data['sequence_number'];
              window.localStorage['isLaid'] = true;
          }).catch(function() {
              console.log("Booo");
          });
          return true;
        }
      }, 4000)

      this.timeout = window.setTimeout(()=>{
        if (window.localStorage['isLaid'] != 'true') {
          const button = document.getElementsByClassName('element')[0]
          button.style.display = 'none'
          label.style.display = 'none'
        }
      }, 4000)

      this.timeout_cover = window.setTimeout(()=>{
        if (this.current == true) {
          const cover = document.getElementsByClassName('cover')[0]
          cover.style.display = 'block'
          cover.style['animation-duration'] = '6s'
          cover.style['animation-name'] = 'uncover'
          this.working = true;
        } else {
          this.doOther();
          console.log('User have laid flower already.')
          const label = document.getElementsByClassName('label')[0]

          fetch('https://p2pebb68te.execute-api.ap-northeast-2.amazonaws.com/park_counter_stage/number', {
            method: 'POST'
          }).then(function(response) {
              return response.json();
          }).then(function(data) {
              console.log(data);
              const button = document.getElementsByClassName('element')[0]
              button.style['visibility'] = 'hidden'
              self.sequence_number = data['sequence_number'];
              label.innerText = `이미 헌화를 해주셨습니다. \n 현재까지 ${self.sequence_number} 명이 헌화해주셨습니다.`
              label.style['animation-duration'] = '8s'
              label.style['animation-name'] = 'labelup'
          }).catch(function() {
              console.log("Booo");
          });
         
        }
      }, 4100);
      this.timeout_cover_2 = window.setTimeout(()=>{
        const cover = document.getElementsByClassName('cover')[0]
        cover.style.display = 'none'
      }, 10100);
      this.timeout3 = window.setTimeout(()=>{
        const framewindow = document.getElementsByClassName('portraitframe')[0].contentWindow;

        const portrait = document.getElementsByClassName('portrait')[0] 
        const frame = document.getElementsByClassName('portraitframe')[0] 
        const wrapper = document.getElementsByClassName('framewrapper')[0] 

        frame.style['width'] = '100%';
        frame.style['height'] = '100%';
        wrapper.style['width'] = '100%';
        wrapper.style['height'] = '100%';

        portrait.style['animation-duration'] = '10s'
        portrait.style['animation-name'] = 'showandhide'
        const screenwidth = window.screen.width
        const screenheight = window.screen.height
        if (screenwidth > screenheight) {
          const width = screenheight - this.offset;
          const height = width * this.ratio;

          portrait.style['width'] = `${width}px`
          portrait.style['height'] = `${height}px`

          const message = {
            wrapperheight: height,
            wrapperwidth: width
          }
          framewindow.postMessage(message, '*')
        } else {
          if (this.portraitsrc.endsWith('200.html')) {
            const width = screenwidth - this.offset * 3;
            const height = width * this.ratio;

            portrait.style['width'] = `${width}px`
            portrait.style['height'] = `${height}px`

            const message = {
              wrapperheight: height,
              wrapperwidth: width
            }
            framewindow.postMessage(message, '*')
          }
          else {
            const width = screenwidth - this.offset;
            const height = width * this.ratio;
  
            portrait.style['width'] = `${width}px`
            portrait.style['height'] = `${height}px`
  
            const message = {
              wrapperheight: height,
              wrapperwidth: width
            }
            framewindow.postMessage(message, '*')
          }
        }
      }, 7500)
      this.timeout4 = window.setTimeout(()=>{
        const portrait = document.getElementsByClassName('portrait')[0]
        portrait.style['display'] = 'none'
      }, 17500)
      this.timeout5 = window.setTimeout(()=>{
        const flower = document.getElementsByClassName('flower')[0]
        const laying = document.getElementsByClassName('laying')[0]
        const flowerimage = document.getElementsByClassName('flowerimage')[0]
        const current_state = this.sequence_number;
        laying.style['display'] = 'flex'
        flower.style['display'] = 'flex'
        laying.style['animation-name'] = 'showup'
        laying.innerText = `${current_state} 번째로 헌화하였습니다.`
        laying.style['animation-duration'] = '5s'
        laying.style['animation-name'] = 'showup'
        flowerimage.style['display'] = 'block'
        flower.style['animation-duration'] = '5s'
        flower.style['animation-name'] = 'showup'
      }, 18500)
      this.timeout6 = window.setTimeout(()=>{
        const laying = document.getElementsByClassName('laying')[0]
        laying.style['opacity'] = '1.0'
        const flower = document.getElementsByClassName('flower')[0]
        flower.style['opacity'] = '1.0'
      }, 22500)
    },
    'doThat': function() {
      console.log('mouseup fired', this.done)
      if (this.working == false) {
        window.clearTimeout(this.timeout)
        window.clearTimeout(this.timeout_cover)
        window.clearTimeout(this.timeout_cover_2)
        window.clearTimeout(this.timeout1)
        window.clearTimeout(this.timeout2)
        window.clearTimeout(this.timeout3)
        window.clearTimeout(this.timeout4)
        window.clearTimeout(this.timeout5)
        window.clearTimeout(this.timeout6)
        const button = document.getElementsByClassName('element')[0]
        button.class = "element convex"
        // button.style.background = 'linear-gradient(-45deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25))';
        button.style['animation-duration'] = '4s'
        button.style['animation-name'] = 'up'
        const label = document.getElementsByClassName('label')[0]
        label.style['animation-duration'] = '4s'
        label.style['animation-name'] = 'labelup'
      }
    },
    'doOther': function() {
      if (this.working == false) {
        window.clearTimeout(this.timeout)
        window.clearTimeout(this.timeout_cover)
        window.clearTimeout(this.timeout_cover_2)
        window.clearTimeout(this.timeout1)
        window.clearTimeout(this.timeout2)
        window.clearTimeout(this.timeout3)
        window.clearTimeout(this.timeout4)
        window.clearTimeout(this.timeout5)
        window.clearTimeout(this.timeout6)
      }
    }
  }
}


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
body {
  align-items: center;
  background: #eeeeee;
  display: flex;
  /*
  display: grid;
  grid-gap: 2rem;
  grid-template-columns: 200px;
  */
  height: 90vh;
  justify-content: center;
}

.cover {
  display: none;
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.0;
  height: 100%;
  width: 100%;
  background: #333333;
}

@keyframes uncover {
  0% {
    opacity: 0.0;
  }
  50% {
    opacity: 1.0;
  }
  100% {
    opacity: 0.0;
  }
}

.portrait {
  display: flex;
  justify-content: center;
  align-items: center;
  opacity: 0.0;
  background: white;
  border: 3rem solid;
  text-align:center;
}

.framewrapper {
  width: 0px;
  height: 0px;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align:center;
}

.portraitframe {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 0px;
  height: 0px;
}

.element {
  align-items: center;
  background: linear-gradient(-45deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
  box-shadow: 12px 12px 16px 0 rgba(0, 0, 0, 0.25),
   -8px -8px 12px 0 rgba(255, 255, 255, 0.3);
  border-radius: 50px;
  display: flex;
  height: 200px;
  justify-content: center;
  width: 200px;
}

.label {
  margin: 30px;
}

.convex {
  background: linear-gradient(-45deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
}

.concave {
  background: linear-gradient(135deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
}

@keyframes down {
  0% {
    background: linear-gradient(-45deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
    opacity: 1.0;
  }
  50% {
    background: linear-gradient(5deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
    opacity: 0.5;
  }
  100% {
    background: linear-gradient(135deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
    opacity: 0.0;
  }
}

@keyframes labeldown {
  0% {
    opacity: 1.0;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    opacity: 0.0;
  }
}

@keyframes up {
  from {
    background: linear-gradient(135deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
    opacity: 0.5;
  }
  to {
    background: linear-gradient(-45deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
    opacity: 1.0;
  }
}

@keyframes labelup {
  from {
    opacity: 0.5;
  }
  to {
    opacity: 1.0;
  }
}

@keyframes showandhide {
  0% {
    opacity: 0.0;
  }
  50% {
    opacity: 1.0;
  }
  70% {
    opacity: 1.0;
  }
  100% {
    opacity: 0.0;
  }
}

.laying {
  margin: 15px;
  padding: 30px;
  align-items: center;
  background: linear-gradient(135deg, rgba(0,0,0,0.22), rgba(255,255,255,0.25));
  box-shadow: 12px 12px 16px 0 rgba(0, 0, 0, 0.25),
   -8px -8px 12px 0 rgba(255, 255, 255, 0.3);
  border-radius: 15px;
  justify-content: center;
  opacity: 0.0;
  display: none;
}

.flower {
  display: none;
  margin: 15px;
  padding: 30px;
  align-items: center;
  justify-content: center;
  opacity: 0.0;
}

.flowerimage {
  display: none;
  width: 200px;
}

@keyframes showup {
  0% {
    opacity: 0.0;
  }
  100% {
    opacity: 1.0;
  }
}

@media (max-width: 600px) {
  body {
    /*
    grid-template-columns: repeat(3, 200px);
    */
  }
  .cover {
    opacity: 0.0;
    height: 100vmax;
    width: 100vmin;
    background: #777777;
  }
  .portrait {
    border: 1rem solid;
  }
}

@media (min-width: 600px) and (max-width: 800px) and (min-height: 600px) and (max-height: 1200px) {
  body {
    /*
    grid-template-columns: repeat(3, 200px);
    */
  }
  .cover {
    opacity: 0.0;
    height: 100vmax;
    width: 100vmin;
    background: #777777;
  }
  .portrait {
    border: 2rem solid;
  }
}

</style>
