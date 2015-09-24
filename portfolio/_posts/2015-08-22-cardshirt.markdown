---
layout:     post
title:      "CardShirt"
image:      cardshirt
tags:       [web]
priority:   2

---
# My Biggest Client So Far

The CardShirt Company is a student run, not-for-profit organization based out of the University of Louisville’s College of Business. The company is dedicated to raising funds for student scholarships and providing students with valuable business experience through the design, marketing, and sale of UofL themed spirit wear and business/organization promotional items. They work to keep students in college financially and teach them the skills they need to transition into a rewarding post-graduate career. Their mission statement is as follows: “The CardShirt Company is a student run, not-for-profit organization based out of the University of Louisville’s College of Business. Here at CardShirt, we are dedicated to raising funds for student scholarships and providing students with valuable business experience through the design, marketing, and sale of UofL themed spirit wear and business/organization promotional items. The CardShirt Company works to keep students in college financially and teach them the skills they need to transition into a rewarding post-graduate career.

I’ve worked with CardShirt since about August 2013 as their Vice President of I.T., which mostly has just entailed redesigning and rebuilding their website. Its most recent iteration not only involved a complete redesign, but also an implementation into Wordpress in order to allow for payment processing through the WooCommerce plugin. Below is some various code and screenshots of the project. Other than the layouts of the files, all of the included code was handwritten – no external libraries such as Bootstrap were used for the design.

# Code

Since the website was built on top of Wordpress, there isn't exactly a whole lot of unique code to post on here. Sure, the design was done by hand as usual, but I never really know how to post snippets of SASS in here. It's all just in a giant file [that you can scroll through yourself][css-file] if you'd like. Luckily for this post, though, there are a few files that I modified enough to be worth showing.

Note: All of this code can be found in the bones theme. Great theme by the way; I would really recommend it to anyone wanting to input their own style without having to write over or delete a mess of pre-done theme stuff.

[front-page.php][front-page]
{% highlight php %}
<?php get_header(); ?>
<div id="content">
    <div class="section store">
        <h1>our products</h1>
        <ul class="products">
            <?php
                $args = array( 'post_type' => 'product', 'stock' => 1, 'posts_per_page' => 3, 'orderby' =>'date','order' => 'DESC' );
                $loop = new WP_Query( $args );
                while ( $loop->have_posts() ) : $loop->the_post(); global $product; ?>
                        <li class="product">
                            <a id="id-<?php the_id(); ?>" href="<?php the_permalink(); ?>" title="<?php the_title(); ?>">
                                <?php if (has_post_thumbnail( $loop->post->ID )) echo get_the_post_thumbnail($loop->post->ID, 'shop_catalog'); else echo '<img src="'.woocommerce_placeholder_img_src().'" alt="Placeholder" width="65px" height="115px" />'; ?>
                                <div class="overlay">
                                    <h3><?php the_title(); ?></h3>
                                    <?php woocommerce_template_loop_add_to_cart( $loop->post, $product ); ?>
                                    <!-- <span class="price"><? //php echo $product->get_price_html(); ?></span> -->
                                </div>
                            </a>
                        </li><!-- /span3 -->
            <?php endwhile; ?>
            <?php wp_reset_query(); ?>
        </ul><!-- /row-fluid -->
        <a href="<?php $page = get_page_by_title('Shop'); echo get_page_link($page->ID); ?>" class="more">Shop All</a>
    </div>
    <div id="hero" class="section hero">
        <div class="wrap">
            <h1>about cardshirt</h1>
            <p>
                The CardShirt Company is a student run, not-for-profit organization based out of the University of Louisville’s College of Business. Here at CardShirt, we are dedicated to raising funds for student scholarships and providing students with valuable business experience through the design, marketing, and sale of UofL themed spirit wear and business/organization promotional items. The Cardshirt Company works to keep students in college financially and teach them the skills they need to transition into a rewarding post-graduate career.
            </p>
            <a class="button" href="<?php echo the_permalink() . '/about/'?>">about us</a>
        </div>
    </div>
    <div class="section content dark">
        <img src="<?php echo get_template_directory_uri(); ?>/library/images/work-background-2.jpg" alt="work-background-2" />
        <div class="wrap">
            <h1>our members</h1>
            <p>
                As a member of CardShirt, you aren't just another face in the crowd. We want you to take on real responsibility, and gain real-world experience that you can apply to your future career. At CardShirt, we're about more than just sitting in our meetings and throwing in your two cents occasionally. As a matter of fact, we'll let you take on as much responsibility as you can handle! Students do it all here - from calling potential business partners to creating and marketing a new product - you can do it all!
            </p>
            <p>
                We are a student-run not-for-profit organization, so we're always looking for new recruits willing to donate some of their time to the CardShirt cause! It’s quite easy to join our ranks. To become involved, <span class="nohref">contact us</span> and let us know you're interested in helping out!
            </p>
        </div>
    </div>
</div>
<?php get_footer(); ?>
{% endhighlight %}

[header.php][header]

{% highlight php %}
<body <?php body_class(); ?> itemscope itemtype="http://schema.org/WebPage">
    <header class="header<?php if ( is_front_page() ) { echo ' static special'; } if ( is_admin_bar_showing() ) { echo ' bump'; } ?>" role="banner" itemscope itemtype="http://schema.org/WPHeader">
        <div class="box left">
            <div class="dropdown">
                <?php
                    $mypages = get_pages();

                    foreach ($mypages as $page) {
                        if ( $page->post_title == 'About' || $page->post_title == 'Shop' ) {
                            echo '<a href="' . get_page_link($page->ID) . '" class="navitem">' . $page->post_title . '</a>';
                        }
                    }
                ?>
                <span class="navitem nohref">Contact</a>
            </div>
            <!-- <div class="dropdown">
                <a class="navitem" href="shop.html">Shop</a>
                <a class="navitem" href="about.html">About</a>
                <span class="navitem nohref">Contact</a>
            </div> -->
        </div>
        <a href="<?php echo home_url(); ?>" rel="nofollow"><img class="logo" src="<?php echo get_template_directory_uri(); ?>/library/images/icon.png"></a>
        <div class="box right">
        <?php
            $cart = WC()->session->get( 'cart', array() );
            $actualCart = WC()->cart;

            if ( is_array( $cart ) && sizeof( $cart ) > 0 ) { ?>
                <span id="itemcount"><?php echo $actualCart->get_cart_contents_count(); ?></span>
                <div class="dropdown">
                    <div class="item th">
                        <span class="cell bold">Shirt Name</span>
                        <span class="cell bold">Quantity</span>
                        <span class="cell bold">Size</span>
                        <span class="cell bold">Price</span>
                    </div>
                <?php
                    foreach ( $cart as $key => $values ) {
                        $_product = wc_get_product( $values['variation_id'] ? $values['variation_id'] : $values['product_id'] );

                        if ( ! empty( $_product ) && $_product->exists() && $values['quantity'] > 0 ) {
                            if ( $_product->is_purchasable() ) { ?>
                                <div class="item">
                                    <span class="cell"><a href="<?php echo $_product->get_permalink(); ?>"><?php echo $_product->post->post_title; ?></a></span>
                                    <span class="cell"><?php echo $values['quantity']; ?></span>
                                    <?php if ( ! empty( $values['variation_id'] ) ) { ?>
                                        <span class="cell"><?php echo ucfirst( current( $values['variation'] ) ); ?></span>
                                    <?php } else { ?>
                                        <span class="cell">n/a</span>
                                    <?php } ?>
                                    <span class="cell">$<?php echo money_format( "%i", ($_product->get_price_excluding_tax() * $values['quantity']) ); ?></span>
                                </div>
                            <?php }
                        }
                    } ?>
                    <div class="item end">
                        <span class="cell">Total: <span class="price"><?php echo $actualCart->get_cart_subtotal(); ?></span></span>
                        <span class="cell"><a href="<?php $page = get_page_by_title('Cart'); echo get_page_link($page->ID); ?>" class="button">Go to Cart</a></span>
                    </div>
                </div>
                <?php } else { ?>
                    <div class="dropdown">
                        <div class="error">
                            <i class="fa fa-warning"></i> Oh no! There are no items in your cart :(
                        </div>
                    </div>
                <?php } ?>
        </div>
    </header>
    <?php if ( is_front_page() ) : ?>
        <div class="beginning">
            <div class="center">
                <img src="<?php echo get_template_directory_uri(); ?>/library/images/cardshirt-logo-white.png" draggable="false">
                <h1>Students Helping Students</h1>
            </div>
            <i class="fa fa-chevron-down fa-2x"></i>
        </div>
        <div class="follow <?php if (is_admin_bar_showing() ) { echo 'margin'; } ?>"></div>
    <?php endif ?>
{% endhighlight %}

[css-file]:     #
[front-page]:   #
[header]:       #