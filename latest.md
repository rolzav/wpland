## Current Setup

### File List:
1. [front-page.php](#front-pagephp)
2. [functions.php](#functionsphp)
3. [includes/enqueue-scripts.php](#includesenqueue-scriptsphp)
4. [includes/modules/ajax-handlers.php](#includesmodulesajax-handlersphp)
5. [includes/modules/custom-post-types.php](#includesmodulescustom-post-typesphp)
6. [assets/js/custom.js](#assetsjscustomjs)

### File Contents:

#### 1. front-page.php:
```php name=front-page.php
<?php
get_header(); ?>

<div class="container-fluid h-100">
    <div class="row h-100">
        <!-- Gallery Section (Left) -->
        <div id="carousel-container" class="col-7 d-flex justify-content-center align-items-center vh-100">
            <div id="demoCarousel" class="carousel slide h-100 w-100 pointer-event" data-bs-ride="carousel">
                <div class="carousel-inner h-100">
                    <?php
                    $gallery = get_post_meta(get_the_ID(), '_demoblock_gallery', true);
                    if (!empty($gallery) && is_array($gallery)) :
                        $first = true;
                        foreach ($gallery as $image_id) : ?>
                            <div class="carousel-item <?php echo $first ? 'active' : ''; ?> h-100">
                                <img src="<?php echo esc_url(wp_get_attachment_url($image_id)); ?>" alt="Gallery Image" class="d-block w-100 h-100 object-fit-cover">
                            </div>
                        <?php $first = false;
                        endforeach;
                    else : ?>
                        <p class="text-center">⚠ No gallery images available.</p>
                    <?php endif; ?>
                </div>
                <button class="carousel-control-prev" type="button" data-bs-target="#demoCarousel" data-bs-slide="prev">
                    <span class="carousel-control-prev-icon" aria-hidden="true"></span>
                </button>
                <button class="carousel-control-next" type="button" data-bs-target="#demoCarousel" data-bs-slide="next">
                    <span class="carousel-control-next-icon" aria-hidden="true"></span>
                </button>
            </div>
        </div>

        <!-- Demo Block List & Details -->
        <div id="demoblock-list-container" class="col-5 d-flex flex-column justify-content-start">
            <div id="demoblock_list">
                <a href="#" id="ensembles-link">ENSEMBLES</a>
                <?php
                $demoblocks = new WP_Query(['post_type' => 'demoblock', 'posts_per_page' => -1]);
                if ($demoblocks->have_posts()) :
                    while ($demoblocks->have_posts()) : $demoblocks->the_post(); ?>
                        <a href="#" data-id="<?php the_ID(); ?>"><?php the_title(); ?></a>
                    <?php endwhile;
                    wp_reset_postdata();
                else : ?>
                    <p><?php _e('No demo blocks found', 'textdomain'); ?></p>
                <?php endif; ?>
            </div>

            <div class="demoblock-details"></div>
        </div>
    </div>
</div>

<div class="container text-center py-4">
    <h2 class="display-5">Message Us</h2>
    <div class="d-flex justify-content-center">
        <div id="message-us">
            <?php echo do_shortcode('[contact-form-7 id="235e694" title="message-us"]'); ?>
        </div>
    </div>
</div>

<?php
get_footer();
?>
```

#### 2. functions.php:
```php name=functions.php
<?php

/**
 * @package Bootscore Child
 *
 * @version 6.0.0
 */

// Exit if accessed directly
defined('ABSPATH') || exit;

// Enqueue scripts and styles
require_once get_stylesheet_directory() . '/includes/enqueue-scripts.php';

// Include additional files from the modules directory
foreach (glob(get_stylesheet_directory() . '/includes/modules/*.php') as $file) {
  require_once $file;
}
```

#### 3. includes/enqueue-scripts.php:
```php name=includes/enqueue-scripts.php
<?php

add_action('wp_enqueue_scripts', 'enqueue_child_styles_scripts');
function enqueue_child_styles_scripts()
{
    $theme_dir = get_stylesheet_directory_uri();
    $theme_path = get_stylesheet_directory();

    // Enqueue styles
    wp_enqueue_style('bootstrap-css', 'https://stackpath.bootstrapcdn.com/bootstrap/5.1.3/css/bootstrap.min.css');
    wp_enqueue_style('main', "$theme_dir/assets/css/main.css", ['parent-style'], date('YmdHi', filemtime("$theme_path/assets/css/main.css")));
    wp_enqueue_style('parent-style', get_template_directory_uri() . '/style.css');
    wp_enqueue_style('custom-css', "$theme_dir/assets/css/custom.css");

    // Enqueue scripts
    wp_enqueue_script('bootstrap-js', 'https://stackpath.bootstrapcdn.com/bootstrap/5.1.3/js/bootstrap.bundle.min.js', ['jquery'], null, true);
    wp_enqueue_script('custom-js', "$theme_dir/assets/js/custom.js", ['jquery'], date('YmdHi', filemtime("$theme_path/assets/js/custom.js")), true);

    // Localize script for AJAX
    wp_localize_script('custom-js', 'ajax_object', ['ajaxurl' => admin_url('admin-ajax.php')]);
}
```

#### 4. includes/modules/ajax-handlers.php:
```php name=includes/modules/ajax-handlers.php
<?php

// AJAX Handler for Content Update
add_action('wp_ajax_load_demoblock', 'load_demoblock_content');
add_action('wp_ajax_nopriv_load_demoblock', 'load_demoblock_content');
function load_demoblock_content()
{
    if (!isset($_POST['post_id']) || !is_numeric($_POST['post_id'])) {
        wp_send_json_error('Invalid request.');
    }

    $post_id = intval($_POST['post_id']);
    $post = get_post($post_id);

    if (!$post) {
        wp_send_json_error('Demo block not found.');
    }

    // Fetch the gallery images
    $gallery = get_post_meta($post_id, '_demoblock_gallery', true);

    ob_start();
    ?>
    <div id="demoCarousel" class="carousel slide h-100 w-100 pointer-event" data-bs-ride="carousel">
        <div class="carousel-inner h-100">
            <?php
            if (!empty($gallery) && is_array($gallery)) :
                $first = true;
                foreach ($gallery as $image_id) : ?>
                    <div class="carousel-item <?php echo $first ? 'active' : ''; ?> h-100">
                        <img src="<?php echo esc_url(wp_get_attachment_url($image_id)); ?>" alt="Gallery Image" class="d-block w-100 h-100 object-fit-cover">
                    </div>
                <?php $first = false;
                endforeach;
            else : ?>
                <p class="text-center">⚠ No gallery images available.</p>
            <?php endif; ?>
        </div>
        <button class="carousel-control-prev" type="button" data-bs-target="#demoCarousel" data-bs-slide="prev">
            <span class="carousel-control-prev-icon" aria-hidden="true"></span>
        </button>
        <button class="carousel-control-next" type="button" data-bs-target="#demoCarousel" data-bs-slide="next">
            <span class="carousel-control-next-icon" aria-hidden="true"></span>
        </button>
    </div>
    <?php
    $gallery_content = ob_get_clean();

    ob_start();
    ?>
    <h2><?php echo esc_html($post->post_title); ?></h2>
    <?php echo apply_filters('the_content', $post->post_content); ?>
    <?php if (has_post_thumbnail($post_id)) : ?>
        <img src="<?php echo esc_url(wp_get_attachment_url(get_post_thumbnail_id($post_id))); ?>" alt="<?php echo esc_attr($post->post_title); ?>" class="img-fluid">
    <?php endif; ?>
    <a href="#message-us" class="btn btn-dark mt-3">Get a Quote</a>
    <?php
    $content = ob_get_clean();

    wp_send_json_success(['gallery' => $gallery_content, 'content' => $content]);
}

// AJAX Handler for ENSAMBLES
add_action('wp_ajax_load_ensembles', 'load_ensembles_content');
add_action('wp_ajax_nopriv_load_ensembles', 'load_ensembles_content');
function load_ensembles_content()
{
    $ensembles = new WP_Query(['post_type' => 'demoblock', 'posts_per_page' => -1]);
    ob_start();
    if ($ensembles->have_posts()) :
        while ($ensembles->have_posts()) : $ensembles->the_post(); ?>
            <div class="col-lg-4 mb-4">
                <a href="#" class="ensembles-item" data-id="<?php the_ID(); ?>">
                    <?php if (has_post_thumbnail()) : ?>
                        <img src="<?php the_post_thumbnail_url('full'); ?>" class="img-fluid" alt="<?php the_title(); ?>">
                    <?php endif; ?>
                </a>
            </div>
        <?php endwhile;
        wp_reset_postdata();
    endif;
    $content = ob_get_clean();

    wp_send_json_success(['content' => $content]);
}
```

#### 5. includes/modules/custom-post-types.php:
```php name=includes/modules/custom-post-types.php
<?php

// Register DEMOBLOCK Custom Post Type
add_action('init', 'register_demoblock_cpt');
function register_demoblock_cpt()
{
  register_post_type('demoblock', [
    'label'         => __('DemoBlock', 'textdomain'),
    'public'        => true,
    'show_in_rest'  => true,
    'menu_icon'     => 'dashicons-grid-view',
    'supports'      => ['title', 'editor', 'thumbnail'],
    'has_archive'   => true,
    'rewrite'       => ['slug' => 'demoblock'],
  ]);
}

// Add gallery meta box to the DEMOBLOCK CPT
add_action('add_meta_boxes', 'add_demoblock_gallery_meta_box');
function add_demoblock_gallery_meta_box()
{
    add_meta_box(
        'demoblock_gallery_meta_box',
        __('DemoBlock Gallery', 'textdomain'),
        'render_demoblock_gallery_meta_box',
        'demoblock',
        'normal',
        'high'
    );
}

function render_demoblock_gallery_meta_box($post)
{
    wp_nonce_field('demoblock_gallery_meta_box_nonce', 'demoblock_gallery_meta_box_nonce');
    $gallery = get_post_meta($post->ID, '_demoblock_gallery', true) ?: [];
    ?>
    <div id="demoblock-gallery-container">
        <?php foreach ($gallery as $image_id) : ?>
            <div class="gallery-image">
                <?php echo wp_get_attachment_image($image_id, 'thumbnail'); ?>
                <input type="hidden" name="demoblock_gallery[]" value="<?php echo esc_attr($image_id); ?>">
                <button class="remove-image">✖</button>
            </div>
        <?php endforeach; ?>
    </div>
    <button id="add-gallery-images" class="button"><?php _e('Add Gallery Images', 'textdomain'); ?></button>

    <script>
    jQuery(document).ready(function($) {
        let frame;
        $('#add-gallery-images').on('click', function(e) {
            e.preventDefault();
            if (frame) frame.open();
            frame = wp.media({
                title: '<?php _e('Select Images', 'textdomain'); ?>',
                multiple: true,
                library: {
                    type: 'image'
                }
            }).on('select', function() {
                let images = frame.state().get('selection').toJSON();
                images.forEach(image => {
                    $('#demoblock-gallery-container').append(`
                        <div class="gallery-image">
                            <img src="${image.sizes.thumbnail.url}" />
                            <input type="hidden" name="demoblock_gallery[]" value="${image.id}">
                            <button class="remove-image">✖</button>
                        </div>
                    `);
                });
            }).open();
        });

        $(document).on('click', '.remove-image', function(e) {
            e.preventDefault();
            $(this).closest('.gallery-image').remove();
        });
    });
    </script>

    <style>
    .gallery-image {
        display: inline-block;
        margin: 5px;
        position: relative;
    }

    .remove-image {
        position: absolute;
        top: 3px;
        right: 3px;
        background: red;
        color: white;
        border: none;
        cursor: pointer;
        font-size: 12px;
        width: 18px;
        height: 18px;
        border-radius: 50%;
    }
    </style>
    <?php
}

add_action('save_post', 'save_demoblock_gallery');
function save_demoblock_gallery($post_id)
{
    if (!isset($_POST['demoblock_gallery_meta_box_nonce']) || !wp_verify_nonce($_POST['demoblock_gallery_meta_box_nonce'], 'demoblock_gallery_meta_box_nonce')) return;
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) return;
    if (!current_user_can('edit_post', $post_id)) return;

    update_post_meta($post_id, '_demoblock_gallery', isset($_POST['demoblock_gallery']) ? array_map('intval', $_POST['demoblock_gallery']) : []);
}
```

#### 6. assets/js/custom.js:
```javascript name=assets/js/custom.js
document.addEventListener("DOMContentLoaded", function () {
  jQuery(document).ready(function ($) {
    function attachEventHandlers() {
      // Handle 'Get a Quote' button click
      $(".go-to-message-us")
        .off("click")
        .on("click", function () {
          var postTitle = $(this).data("title");
          $("#post-title").val(postTitle);
          $("#your-subject").val(postTitle);
        });

      // Handle "ENSEMBLES" link click
      $("#ensembles-link").on("click", function (e) {
        e.preventDefault();
        
        $.ajax({
          url: ajax_object.ajaxurl,
          type: "POST",
          data: {
            action: "load_ensembles"
          },
          success: function (response) {
            if (response.success) {
              $(".demoblock-details").html(response.data.content);
              attachEventHandlers();
            } else {
              alert("Failed to load content.");
            }
          },
        });
      });

      // Handle demoblock link click
      $("#demoblock_list a").not("#ensembles-link").click(function (e) {
        e.preventDefault();
        var postID = $(this).data("id");

        $.ajax({
          url: ajax_object.ajaxurl,
          type: "POST",
          data: {
            action: "load_demoblock",
            post_id: postID,
          },
          success: function (response) {
            if (response.success) {
              // Update the gallery (left col-7)
              $("#carousel-container").html(response.data.gallery);
              
              // Update the content (right col-5)
              $(".demoblock-details").html(response.data.content);

              // Reinitialize the carousel
              $("#demoCarousel").carousel();

              // Reattach event handlers for the new content
              attachEventHandlers();
            } else {
              alert("Failed to load content.");
            }
          },
        });
      });

      // Handle ensembles item click
      $(document).on("click", ".ensembles-item", function(e) {
        e.preventDefault();
        var postID = $(this).data("id");

        $.ajax({
          url: ajax_object.ajaxurl,
          type: "POST",
          data: {
            action: "load_demoblock",
            post_id: postID,
          },
          success: function (response) {
            if (response.success) {
              // Update the gallery (left col-7)
              $("#carousel-container").html(response.data.gallery);
              
              // Update the content (right col-5)
              $(".demoblock-details").html(response.data.content);

              // Reinitialize the carousel
              $("#demoCarousel").carousel();

              // Reattach event handlers for the new content
              attachEventHandlers();
            } else {
              alert("Failed to load content.");
            }
          },
        });
      });
    }

    // Attach initial event handlers
    attachEventHandlers();
  });
});
```
