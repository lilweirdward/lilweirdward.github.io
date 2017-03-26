---
layout:     post
title:      "CARDS Fest"
image:      cardsfest
tags:       [web]
priority:   1
author:     Jecorey Arthur
authorlink: http://cardsfest.cardshirt.org

---
# UofL's Biggest Music Festival

At the beginning of the summer of 2015, I was approached by the President of CardShirt about a new idea that he had started coming up with: a festival to celebrate the music, food, and culture of Louisville on its own university campus. Obviously, I suggested and was approved to build the website to be used to promote the event.

I had several takeaways from this project - some of them being pretty unique when compared to anything that I'd done previously. The biggest thing that I proved to myself was that I could work well with a client independently. This was something that I attempted earlier in my school tenure, and my lack of maturity created a pretty poor relationship that I still have regrets about. Between deadlines and evolving requirements that were all met, I was able to justify that I had matured over the years. 

As with almost everything that I've worked on, there are aspects of the website that I love and hate. Probably the second most important thing that I did with this project was building a CRUD tool that was supposed to allow the two event organizers to update the event info without going through me. I went through a few Angular.JS tutorials, read a couple articles about writing a PHP API, and spent nearly 72 straight hours trying to put this CRUD tool together. Unfortunately, I failed miserably. I got all the way to getting the API to actually work (albeit with horrible security), and when I went to hook the Angular up to it, the API crashed on me. Being a 3rd straight day into the coding with only a day left to wrap the project up (yes, those 3 days were also the weekend before the go-live date), I bailed on the idea and just wrote the website statically. I really regret not planning ahead more and getting this feature complete, since it would've been my biggest development effort of the year.

# Code

I'll start off with a link to [the CRUD tool][crud-tool]. I actually wrestled for a while with whether or not to actually post snippets of the code on here, since I'll probably just give away how terrible of a PHP dev I am. Then I realized that it's already on GitHub anyway, and you might as well at least see that I tried.

[index.php][index-php]
{% highlight php %}
<?php
  include_once 'dbconfig.php';
  
  if (isset($_GET['delete_artists_id']))
  {
    $sql_query = "DELETE FROM artists WHERE id = ".$_GET['delete_artists_id'];
    $mysqli->query($sql_query);
    header("Location: $_SERVER[PHP_SELF]");
  }
  ...
?>
<!DOCTYPE html>
<html>
<head>
  ...
  <script>
    function edta_id(id) {
      window.location.href = 'add-artists-data.php?edit_id='+id;
    }
    function deletea_id(id) {
      if (confirm('Are you sure you want to delete this artist?')) {
        window.location.href = 'index.php?delete_artists_id='+id;
      }
    }
    ...
  </script>
</head>
<body>
    <h1>Artists</h1>
    <a href="add-artists-data.php">Add new record(s)</a>
    <table border="1">
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Title</th>
            <th>Description</th>
            <th>Soundcloud URL</th>
      <th>Actions</th>
        </tr>
    <?php
      $sql_query = "SELECT * FROM artists";
      $result_set = $mysqli->query($sql_query);
      while ($row = $result_set->fetch_assoc())
      {
    ?>
        <tr>
            <td><?php echo $row['id']; ?></td>
            <td><?php echo $row['name']; ?></td>
            <td><?php echo $row['title']; ?></td>
            <td><?php echo $row['description']; ?></td>
            <td class="url">*click edit to see*</td>
        <td><a href="javascript:edta_id('<?php echo $row['id']; ?>')">Edit</a> | <a href="javascript:deletea_id('<?php echo $row['id']; ?>')">Delete</a></td>
        </tr>
    <?php
      }
    ?>
    </table>
    ...
 </body>
{% endhighlight %}

[dbconfig.php][dbconfig]
{% highlight php %}
<?php
    $mysqli = new mysqli("...", "...", "...", "...");
    if ($mysqli->connect_errno) {
        echo "Failed to connect to MySQL: (" . $mysqli->connect_errno . ") " . $mysqli->connect_error;
    }
    echo $mysqli->host_info . "\n";
?>
{% endhighlight %}

[api.class.php][api-class]
{% highlight php %}
<?php
...
/**
 * Constructor: __construct
 * Allow for CORS, assemble and pre-process the data
 */
 public function __construct($request) {
    header("Access-Control-Allow-Origin: *");
    header("Access-Control-Allow-Methods: *");
    header("Content-Type: application/json");
    
    $this->args = explode('/', rtrim($request, '/'));
    $this->endpoint = array_shift($this->args);
    if (array_key_exists(0, $this->args) && !is_numeric($this->args)) {
        $this->verb = array_shift($this->args);
    }
    
    $this->method = $_SERVER['REQUEST_METHOD'];
    // if ($this->method == 'POST' && array_key_exists('HTTP_X_HTTP_METHOD', $_SERVER)) {
    //  if ($_SERVER['HTTP_X_HTTP_METHOD'] == 'DELETE') {
    //      $this->method = 'DELETE';
    //  } else if ($_SERVER['HTTP_X_HTTP_METHOD'] == 'PUT') {
    //      $this->method = 'PUT';
    //  } else {
    //      throw new Exception("Unexpected Header");
    //  }
    //  }
    
    switch($this->method) {
        case 'GET':
            $this->request = $this->_cleanInputs($_GET);
            break;
        default:
            $this->_response('Invalid method', 405);
            break;
    }
 }
 ...
 ?>
{% endhighlight %}

For the rest of the terrible PHP, feel free to visit [the GitHub repository][github-crud]. Also, if you happen to know what's wrong with it, or just want to tell me how horribly I write PHP, feel free to [tweet at me][twitter].

The only other part of this project was the front end stuff. To be honest, I'm not really sure if there's much code worth posting here, since it's mostly either pretty standard HTML or pretty below-average SASS. Obviously, you can also [visit that GitHub repo][github-main] if you really want to see. Just for the sake of having code though, here's the SASS that I wrote for the sandwich bar icon.

{% highlight css %}
.lines-button {
    display: inline-block;
    position: absolute;
    top: 0; right: 0;
    height: 4rem;
    width: 4rem;
    @include transition(all $transition);
    cursor: pointer;
    user-select: none;
    background-color: $gray;
    z-index: 1;
    &:hover {
        background-color: $black;
        opacity: 1;
        .lines {
            &:before { top: 0.35rem; background: white; }
            &:after { top: -0.35rem; background: white; }
        }
    }
    &:active {
        transition: 0;
    }
    &:focus {
        outline: none;
    }
    &.x.close {
        background-color: $black;
        .lines {
            /*overlay the lines by setting both their top values to 0*/
            &:before, &:after{
                transform-origin: 50% 50%;
                top:0;
                width: $button-size;
                background: white;
            }
            // rotate the lines to form the x shape
            &:before{
                transform: rotate3d(0,0,1,45deg); 
            }
            &:after{
                transform: rotate3d(0,0,1,-45deg); 
            }
        }
        & ~ ul.navitems {
            display: block;
        }
    }
    .lines {
        //create middle line
        @include line;
        position: relative;
        /*hide the middle line*/
        background: transparent; 
        /*create the upper and lower lines as pseudo-elements of the middle line*/
        &:before, &:after {
            @include line;
            position: absolute;
            left:0;
            content: '';
            transform-origin: 0.125rem center;
        }
        &:before { top: 0.25rem; }
        &:after { top: -0.25rem; }
    }
}
{% endhighlight %}

[crud-tool]:    http://crud.www.cardshirt.org/
[dbconfig]:     #
[index-php]:    #
[api-class]:    #
[github-crud]:  https://github.com/lilweirdward/cardsfest-crud
[twitter]:      https://twitter.com/lilweirdward
[github-main]:  https://github.com/lilweirdward/cardsfest