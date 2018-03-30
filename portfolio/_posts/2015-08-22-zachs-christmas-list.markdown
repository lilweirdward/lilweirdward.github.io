---
layout:     post
title:      "Zach's Christmas List"
image:      christmas
tags:       [web]
priority:   4

---
# All I Want for Christmas

We all make physical Christmas lists as kids, right? We probably hand wrote most of them once upon a time, and then when technology started getting more prominent, my siblings and I usually just typed them up really quickly in a Word document. As I got older and started learning more about computers and whatnot, I started seeing how fancy of Christmas lists I could make – stuff like pictures of my items and the names of places they could be bought were common inclusions. I probably could be more mature these days and not make physical Christmas lists anymore, or I could see it as an opportunity to test how much I’ve learned over the past year on developing websites. Yeah, [I picked the second option](http://www.zachschristmaslist.com) for the past two years.

# Code
    
On top of my usual formula of using HTML and SASS to build my website, I also used KnockoutJS to practice fetching a JSON array and loop over it to build sections of [the website](https://github.com/lilweirdward/zachschristmaslist). Of course, this ended up being completely unnecessary in the context of a website with no database or changing list of items, and actually made things more complicated than they needed to be. It made for some great practice in learning some of the ins and outs of the library though, which ultimately justified those immature feelings over still making a Christmas list.

[index.html](https://github.com/lilweirdward/zachschristmaslist/blob/gh-pages/index.html)
{% highlight html %}
...
<div class="blackcover" data-bind="with: selectedItem">
    <div class="product">
        <img data-bind="attr: { src: imageSrc }" class="item">
        <div class="contained">
            <h1 id="product_name" data-bind="text: name"></h1>
            <a href="#" class="button"><span data-bind="text: price"></span></a>
            <p>
                <em>Description:</em> <span data-bind="html: description"></span>
            </p>
            <h2>Interest Level</h2>
            <div class="progress-wrap" data-bind="attr: { 'data-progress-percent': interestScore, title: interestScore() + '/10' }">
                <div class="progress-bar"></div>
            </div>
        </div>
        <span class="close">x</span>
    </div>
</div>
...
<div id="gadgets" class="category" data-bind="foreach: gadgets, afterAllRendered: gadgetsFinished">
    <div class="wrapper">
        <div class="item">
            <img data-bind="attr: { src: imageSrc }, click: $parent.selectItem">
            <h2 data-bind="text: name"></h2>
            <p>
                Price: <span data-bind="text: price"></span>
            </p>
        </div>
    </div>
</div>
...
<script type="text/javascript">
    function Item(data) {
        this.id = ko.observable(data.id);
        this.category = ko.observable(data.category);
        this.name = ko.observable(data.name);
        this.imageSrc = ko.observable(data.imageSrc);
        this.price = ko.observable(data.price);
        this.interestScore = ko.observable(data.interestScore);
        this.description = ko.observable(data.description);
    }

    function ChristmasListViewModel() {
        var self = this;
        self.gadgets = ko.observableArray([]);
        self.clothes = ko.observableArray([]);
        self.misc = ko.observableArray([]);
        self.selectedItem = ko.observable();
        var gadgetsUri = "https://rawgit.com/bonso/bonso.github.io/master/scripts/gadgets.json";
        var clothesUri = "https://rawgit.com/bonso/bonso.github.io/master/scripts/clothes.json";
        var miscUri = "https://rawgit.com/bonso/bonso.github.io/master/scripts/misc.json";

        self.selectItem = function(item) {
            self.selectedItem(item);
            $('.blackcover').addClass('on');
            $('.product span.close').click(function() {
                $('.blackcover').removeClass('on');
            });
            moveProgressBar();
            $(window).resize(function() {
                moveProgressBar();
            });
        }

        self.gadgetsFinished = function(value) {
            var container = document.querySelector('#gadgets');
            var msnry;
            // initialize Masonry after all images have loaded
            imagesLoaded( container, function() {
                msnry = new Masonry( container );
            });
        }

        self.clothesFinished = function(value) {
            var container = document.querySelector('#clothes');
            var msnry;
            // initialize Masonry after all images have loaded
            imagesLoaded( container, function() {
                msnry = new Masonry( container );
            });
        }

        self.miscFinished = function(value) {
            var container = document.querySelector('#misc');
            var msnry;
            // initialize Masonry after all images have loaded
            imagesLoaded( container, function() {
                msnry = new Masonry( container );
            });
        }

        $.getJSON(gadgetsUri, function(data) {
            var tempArray = [];
            $.each(data, function(i) {
                tempArray.push(new Item(data[i]));
            });
            self.gadgets(tempArray);
        });

        $.getJSON(clothesUri, function(data) {
            var tempArray = [];
            $.each(data, function(i) {
                tempArray.push(new Item(data[i]));
            });
            self.clothes(tempArray);
        });

        $.getJSON(miscUri, function(data) {
            var tempArray = [];
            $.each(data, function(i) {
                tempArray.push(new Item(data[i]));
            });
            self.misc(tempArray);
        });
    }

    ko.bindingHandlers.afterAllRendered = {
        update: function(el, va, ab) {
            ab().foreach && va()(ab().foreach());
        }
    }

    ko.applyBindings(new ChristmasListViewModel());

    function moveProgressBar() {
        var getPercent = (($('.progress-wrap').data('progress-percent') * 10) / 100);
        var getProgressWrapWidth = $('.progress-wrap').width();
        var progressTotal = getPercent * getProgressWrapWidth;
        $('.progress-bar').stop().animate({
            left: progressTotal
        }, 500);
    }

    $(document).ready(function() {
        if ($('html').hasClass('ie')) {
            window.location.replace('ie.html');
        }

        $.fn.snow({ newOn: 300 });

        // $(".category").mousewheel(function(event) {
        //  if (event.originalEvent.wheelDelta >= 0) {
        //      if ($(this).scrollLeft() > 0) {
        //          this.scrollLeft -= (event.deltaY * event.deltaFactor);
        //          event.preventDefault();
        //      }
        //  }
        //  else {
        //      if ($(this).scrollLeft() < (this.scrollWidth - $(this).width())) {
        //          this.scrollLeft -= (event.deltaY * event.deltaFactor);
        //          event.preventDefault();
        //      }
        //  }
        // });
    });
</script>
{% endhighlight %}