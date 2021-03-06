## waterfall-sinanews
预览地址:https://aledenn.github.io/waterfall-sinanews/index
采用的技术如下：
## 懒加载原理
1. 在网页最下面有一个load盒子,当滚动到loading出现在可视窗口内，发送ajax请求，获得数据后，加载数据。
```
<div id="load">loading...</div>
```
判断是否出现在页面中：
```
  isShow: function( $el) {

    var scrollH = this.$c.scrollTop(),  //滚动的高度
      winH = this.$c.height(),  //窗口的高度
      top = $el.offset().top; //在文档留中的位置

    if (top < winH + scrollH) {
      return true;
    } else {
      return false;
    }
  }
```
函数节流 -- 避免滚动事件多次触发，发送多个ajax请求
```
    bind: function() {

    var me = this,
      timer = null,
      interval = 100;

    $(window).on('scroll', function(e) {
      timer && clearTimeout(timer);
      timer = setTimeout(function() {
        me.checkShow();
      }, interval);
    });

  },
```
## 瀑布流原理
1. 计算每一行能容纳多少列，并初始化arrColHeight
2. 使用arrColHeight记录每一列的高度
3. 找到最短的一列，插入items
4. 修改arrColHeight数组

初始化arrColHeight数组
```
   this.$items = this.$ct.find('.item');
    if(this.$items.length ===0) return;
    this.itemWidth = this.$items.outerWidth(true);
    this.$ct.width('auto');
    this.colNum = Math.floor( this.$ct.width() / this.itemWidth );  //计算列数
    this.$ct.width(this.itemWidth*this.colNum);
    if(this.arrColHeight.length === 0 || !$nodes){   // 初始化arrColHeight
      this.arrColHeight = [];
      for(var i=0; i<this.colNum; i++){
        this.arrColHeight[i] = 0;
      }	
    }
```
放置元素
```
  placeItem: function( $el ) {
    // 1. 找到arrColHeight的最小值，得到是第几列
    // 2. 元素left的值是 列数*宽度
    // 3. 元素top的值是 最小值
    // 4. 放置元素的位置，把arrColHeight对应的列数的值加上当前元素的高度
    var obj = this.getIndexOfMin(this.arrColHeight),  // getIndexofMin获得arrColHeight中的最小值
      idx = obj.idx,
      min = obj.min;
    $el.css({
      left: idx * this.itemWidth,
      top: min
    });
    this.arrColHeight[idx] += $el.outerHeight(true);
  }
```
## 实现原理
ajax请求数据
```
	$.ajax({
		url: 'https://platform.sina.com.cn/slide/album_tech',
		type: 'get',
		dataType: 'jsonp',
		jsonp:"jsoncallback",
		data: {
			app_key: '1271687855',
			format:'json',    
			size:'img',
			num: perPageCount,
			page: curPage
		},
		success: function(ret){
			if(ret.status.code == 0){
				callback(ret.data);
				curPage++;
				perPageCount=10
			}
		}
	})
}
```
封装了renderData用于页面插入数据
```
function renderData(items)
```