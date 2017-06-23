# swiper
a simple swiper for vue


## install
> npm install banner-swiper --save-dev

## usage

    import Vue from 'vue'
    import Swiper from 'banner-swiper'

    new Vue({
      el: 'body',
      components: {Swiper}
    });
    
    
    
    <swiper>
      <div class="swiper-item">1</div>
      <div class="swiper-item">2</div>
      <div class="swiper-item">3</div>
    </swiper>
    
    
## API
    
| Name           | default       | description              |
| -------------  |:-------------:| -----:                   |
| loop           | ture          | auto play                |
| speed          | 3000          | the speed of playing     |
| showIndicators | ture          | show this indicators     |
| height         | 250px         | define div height        |
| indicatorType  | 1             | define indicator position|


## for example

<swiper :height="'540px'" :indidcatorType="2"></swiper>

set the height of swiper to 540px and set the indicatorType to the lower right corner

the indicatorType's default is in the middle

