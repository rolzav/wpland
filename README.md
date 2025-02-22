File List:
front-page.php
functions.php
includes/enqueue-scripts.php
includes/modules/ajax-handlers.php
includes/modules/custom-post-types.php
includes/modules/meta-boxes.php
assets/js/custom.js
assets/scss/aaa_bootscore-custom.scss
File Contents:
1. front-page.php:
<?php
get_header(); ?>

<div class="container-fluid h-100">
    <div class="row h-100">
        <!-- Gallery Section (Left) -->
        <div class="col-8 d-flex justify-content-center align-items-center vh-100">
            <div id="modulesCarousel" class="carousel slide h-100 w-100" data-bs-ride="carousel">
                <div class="carousel-inner h-100">
                    <?php
                    $gallery = get_post_meta(get_the_ID(), '_modulebox_gallery', true);
                    $gallery = maybe_unserialize($gallery); // Ensure correct format

                    if (!empty($gallery) && is_array($gallery)) :
                        $first = true;
                        foreach ($gallery as $image_id) : ?>
                    <div class="carousel-item <?php echo $first ? 'active' : ''; ?> h-100">
                        <img src="<?php echo esc_url(wp_get_attachment_url($image_id)); ?>" alt="Featured Image"
                            class="d-block w-100 h-100 object-fit-cover featured-image">
                    </div>
                    <?php $first = false;
                        endforeach;
                    else : ?>
                    <p class="text-center">⚠ No gallery images available.</p>
                    <?php endif; ?>
                </div>
                <button class="carousel-control-prev" type="button" data-bs-target="#modulesCarousel"
                    data-bs-slide="prev">
                    <span class="carousel-control-prev-icon" aria-hidden="true"></span>
                </button>
                <button class="carousel-control-next" type="button" data-bs-target="#modulesCarousel"
                    data-bs-slide="next">
                    <span class="carousel-control-next-icon" aria-hidden="true"></span>
                </button>
            </div>
        </div>

        <!-- Module List & Details -->
        <div class="col-4 d-flex flex-column justify-content-start">
            <div id="modulebox_list">
                <?php
                $modules = new WP_Query(['post_type' => 'modulebox', 'posts_per_page' => -1]);
                if ($modules->have_posts()) :
                    while ($modules->have_posts()) : $modules->the_post(); ?>
                <a href="#" data-id="<?php the_ID(); ?>"><?php the_title(); ?></a>
                <?php endwhile;
                    wp_reset_postdata();
                endif;
                ?>
            </div>

            <div class="module-details"></div>
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

2. functions.php:
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

3. includes/enqueue-scripts.php:
<?php

add_action('wp_enqueue_scripts', 'bootscore_child_enqueue_styles');
function bootscore_child_enqueue_styles()
{
    $theme_dir = get_stylesheet_directory_uri();
    $theme_path = get_stylesheet_directory();

    // Enqueue styles
    wp_enqueue_style('main', "$theme_dir/assets/css/main.css", ['parent-style'], date('YmdHi', filemtime("$theme_path/assets/css/main.css")));
    wp_enqueue_style('parent-style', get_template_directory_uri() . '/style.css');
    wp_enqueue_style('lightbox-css', "$theme_dir/assets/css/styles.css");

    // Enqueue scripts
    wp_enqueue_script('custom-js', "$theme_dir/assets/js/custom.js", ['jquery'], date('YmdHi', filemtime("$theme_path/assets/js/custom.js")), true);
    wp_enqueue_script('lightbox-js', "$theme_dir/assets/js/scripts.js", ['jquery'], null, true);
}

/**
 * Enqueue AJAX script for Modulebox
 */
add_action('wp_enqueue_scripts', 'enqueue_ajax_script');
function enqueue_ajax_script()
{
    wp_enqueue_script('modulebox-ajax', get_stylesheet_directory_uri() . '/js/modulebox-ajax.js', ['jquery'], null, true);
    wp_localize_script('modulebox-ajax', 'ajax_object', ['ajaxurl' => admin_url('admin-ajax.php')]);
}

4. includes/modules/ajax-handlers.php:
<?php

// AJAX Handler for Content Update
add_action('wp_ajax_load_modulebox', 'load_modulebox_content');
add_action('wp_ajax_nopriv_load_modulebox', 'load_modulebox_content');
function load_modulebox_content()
{
    if (!isset($_POST['module_id']) || !is_numeric($_POST['module_id'])) {
        wp_send_json_error('Invalid request.');
    }

    $post_id = intval($_POST['module_id']);
    $post = get_post($post_id);

    if (!$post) {
        wp_send_json_error('Module not found.');
    }

    // Fetch the gallery images
    $gallery = get_post_meta($post_id, '_modulebox_gallery', true);
    $gallery = maybe_unserialize($gallery);

    ob_start();
    if (!empty($gallery) && is_array($gallery)) :
        $first = true;
        foreach ($gallery as $image_id) : ?>
<div class="carousel-item <?php echo $first ? 'active' : ''; ?>">
    <img src="<?php echo esc_url(wp_get_attachment_url($image_id)); ?>" alt="Featured Image"
        class="d-block w-100 h-100 object-fit-cover featured-image">
</div>
<?php
            $first = false;
        endforeach;
    endif;
    $gallery_content = ob_get_clean();

    ob_start();
    ?>
<h2><?php echo esc_html($post->post_title); ?></h2>
<?php echo apply_filters('the_content', $post->post_content); ?>
<?php if (has_post_thumbnail($post_id)) : ?>
<a href="<?php echo esc_url(wp_get_attachment_url(get_post_thumbnail_id($post_id))); ?>" class="featured-image">
    <?php echo get_the_post_thumbnail($post_id, 'full'); ?>
</a>
<?php endif; ?>
<a href="#message-us" class="btn btn-dark mt-3" data-title="<?php echo esc_attr($post->post_title); ?>">Go to Message
    Us</a>
<?php
    $content = ob_get_clean();

    wp_send_json_success(['gallery' => $gallery_content, 'content' => $content]);
}

5. includes/modules/custom-post-types.php:
<?php

// Register MODULEBOX Custom Post Type
add_action('init', 'register_modulebox_cpt');
function register_modulebox_cpt()
{
  register_post_type('modulebox', [
    'label'         => __('ModuleBox', 'textdomain'),
    'public'        => true,
    'show_in_rest'  => true,
    'menu_icon'     => 'dashicons-grid-view',
    'supports'      => ['title', 'editor', 'thumbnail'],
    'has_archive'   => false,
    'rewrite'       => ['slug' => 'modulebox'],
  ]);
}

6. includes/modules/meta-boxes.php:
<?php

// Add Gallery Functionality in the Editor
add_action('add_meta_boxes', 'modulebox_gallery_meta_box');
function modulebox_gallery_meta_box()
{
    add_meta_box('modulebox_gallery', __('Module Gallery', 'textdomain'), 'modulebox_gallery_callback', 'modulebox', 'normal', 'high');
}

function modulebox_gallery_callback($post)
{
    wp_nonce_field('modulebox_gallery_nonce', 'modulebox_gallery_nonce_field');
    $gallery = get_post_meta($post->ID, '_modulebox_gallery', true) ?: [];
?>
<div id="modulebox-gallery-container">
    <?php foreach ($gallery as $image_id) : ?>
    <div class="gallery-image">
        <?php echo wp_get_attachment_image($image_id, 'thumbnail'); ?>
        <input type="hidden" name="modulebox_gallery[]" value="<?php echo esc_attr($image_id); ?>">
        <button class="remove-image">✖</button>
    </div>
    <?php endforeach; ?>
</div>
<button id="add-gallery-images" class="button"><?php _e('Add the Gallery', 'textdomain'); ?></button>

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
                $('#modulebox-gallery-container').append(`
                    <div class="gallery-image">
                        <img src="${image.sizes.thumbnail.url}" />
                        <input type="hidden" name="modulebox_gallery[]" value="${image.id}">
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

add_action('save_post', 'save_modulebox_gallery');
function save_modulebox_gallery($post_id)
{
    if (!isset($_POST['modulebox_gallery_nonce_field']) || !wp_verify_nonce($_POST['modulebox_gallery_nonce_field'], 'modulebox_gallery_nonce')) return;
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) return;
    if (!current_user_can('edit_post', $post_id)) return;

    update_post_meta($post_id, '_modulebox_gallery', isset($_POST['modulebox_gallery']) ? array_map('intval', $_POST['modulebox_gallery']) : []);
}

7. assets/js/custom.js:
document.addEventListener("DOMContentLoaded", function () {
    jQuery(document).ready(function ($) {
      function attachEventHandlers() {
        // Handle 'Go to Message Us' button click
        $(".go-to-message-us")
          .off("click")
          .on("click", function () {
            var postTitle = $(this).data("title");
            $("#post-title").val(postTitle);
            $("#your-subject").val(postTitle);
          });
      }
  
      // Attach initial event handlers
      attachEventHandlers();
  
      // Handle module link click
      $("#modulebox_list a").click(function (e) {
        e.preventDefault();
        var postID = $(this).data("id");
  
        $.ajax({
          url: ajax_object.ajaxurl,
          type: "POST",
          data: {
            action: "load_modulebox",
            module_id: postID,
          },
          success: function (response) {
            if (response.success) {
              // Update the gallery (left col-7)
              $("#modulesCarousel .carousel-inner").html(response.data.gallery);
  
              // Update the content (right col-5)
              $(".module-details").html(response.data.content);
  
              // Reinitialize the carousel
              $("#modulesCarousel").carousel();
  
              // Reattach event handlers for the new content
              attachEventHandlers();
            } else {
              alert("Failed to load content.");
            }
          },
        });
      });
    });
  });
  
8. assets/scss/aaa_bootscore-custom.scss:
.carousel-item {
    height: 100%;
  }
  
  .carousel-item img {
    object-fit: cover;
    height: 100%;
    width: 100%;
  }
  
  .module-details {
    img {
      width: 248px;
    }
  }
  
  .module-details {
    display: flex;
    flex-direction: column;
  
    img {
      width: 248px;
    }
  
    .btn {
      width: 248px;
      padding: 20px;
    }
  }
  
  .carousel-item {
    height: 100%;
  }
  
  .carousel-item img {
    object-fit: cover;
    height: 100%;
    width: 100%;
  }
  
  .lightbox {
    display: none;
    position: fixed;
    z-index: 9999;
    padding-top: 60px;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.9);
  }
  
  .lightbox-content {
    margin: auto;
    display: block;
    width: 80%;
    max-width: 700px;
  }
  
  .close {
    position: absolute;
    top: 15px;
    right: 35px;
    color: #fff;
    font-size: 40px;
    font-weight: bold;
    transition: 0.3s;
    cursor: pointer;
  }
  
  .close:hover,
  .close:focus {
    color: #bbb;
    text-decoration: none;
    cursor: pointer;
  }

Each file's content is presented within code blocks, making it easy to read and understand.


