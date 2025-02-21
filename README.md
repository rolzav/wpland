Hello, Copilot!

# SETUP:
- WordPress installed
- Parent theme Bootscore
- Child theme of Bootscore
- front-page.php created.
- modulebx-ajax.js

So far this is setup.

# WHAT IS NEEDED?

I am trying to achieve to have Modulebox cpt (already added in functions.php please check)
Then need to add gallery (WP pre-built-in functionality, that handles well and saves correctly)
Then on the front-page.php Modulebox section where is container>row>col-7 and col-5
where, col-7 is the gallery (ideally if bootstrap carousel) and, col-5 is already set correctly I think so, where is necessary to display all the title of CPT as urls and then Title, Content, and featured image and button.)
Button triggers to form.


Please check if all is good or no, if some code or anything is missing or incorrect, please clarify and look for how could it be fixed the right way!


# Here is the files I have so far:

########## FRONT-PAGE.PHP FILE CODE:
<?php
get_header(); ?>

<div class="container-fluid h-100">
    <div class="row h-100">
        <!-- Gallery Section (Left) -->
        <div class="col-7 d-flex justify-content-center align-items-center">
            <div id="modulesCarousel" class="carousel slide" data-bs-ride="carousel">
                <div class="carousel-inner">
                    <?php
                    $gallery = get_post_meta(get_the_ID(), '_modulebox_gallery', true);
                    $gallery = maybe_unserialize($gallery); // ✅ Ensure correct format

                    if (!empty($gallery) && is_array($gallery)) :
                        $first = true;
                        foreach ($gallery as $image_id) : ?>
                    <div class="carousel-item <?php echo $first ? 'active' : ''; ?>">
                        <a href="<?php echo esc_url(wp_get_attachment_url($image_id)); ?>"
                            data-lightbox="modules-carousel">
                            <?php echo wp_get_attachment_image($image_id, 'full'); ?>
                        </a>
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
        <div class="col-5 d-flex flex-column justify-content-center">
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


########## FUNCTIONS.PHP FILE CODE:
<?php

/**
 * @package Bootscore Child
 *
 * @version 6.0.0
 */


// Exit if accessed directly
defined('ABSPATH') || exit;


/**
 * Enqueue scripts and styles
 */
add_action('wp_enqueue_scripts', 'bootscore_child_enqueue_styles');
function bootscore_child_enqueue_styles()
{

  // Compiled main.css
  $modified_bootscoreChildCss = date('YmdHi', filemtime(get_stylesheet_directory() . '/assets/css/main.css'));
  wp_enqueue_style('main', get_stylesheet_directory_uri() . '/assets/css/main.css', array('parent-style'), $modified_bootscoreChildCss);

  // style.css
  wp_enqueue_style('parent-style', get_template_directory_uri() . '/style.css');

  // custom.js
  // Get modification time. Enqueue file with modification date to prevent browser from loading cached scripts when file content changes. 
  $modificated_CustomJS = date('YmdHi', filemtime(get_stylesheet_directory() . '/assets/js/custom.js'));
  wp_enqueue_script('custom-js', get_stylesheet_directory_uri() . '/assets/js/custom.js', array('jquery'), $modificated_CustomJS, false, true);
}

function enqueue_ajax_script()
{
    wp_enqueue_script('modulebox-ajax', get_stylesheet_directory_uri() . '/js/modulebox-ajax.js', array('jquery'), null, true);

    wp_localize_script('modulebox-ajax', 'ajax_object', [
        'ajaxurl' => admin_url('admin-ajax.php'),
    ]);
}
add_action('wp_enqueue_scripts', 'enqueue_ajax_script');




// 1️⃣ Register MODULEBOX Custom Post Type
function register_modulebox_cpt()
{
  $args = [
    'label'         => __('ModuleBox', 'textdomain'),
    'public'        => true,
    'show_in_rest'  => true, // Enables REST API support
    'menu_icon'     => 'dashicons-grid-view',
    'supports'      => ['title', 'editor', 'thumbnail'],
    'has_archive'   => false,
    'rewrite'       => ['slug' => 'modulebox'],
  ];
  register_post_type('modulebox', $args);
}
add_action('init', 'register_modulebox_cpt');



// 2️⃣ Add Gallery Functionality in the Editor
// Add "Add the Gallery" Button
// Add this in functions.php to create a meta box for gallery images:
function modulebox_gallery_meta_box()
{
  add_meta_box(
    'modulebox_gallery',
    __('Module Gallery', 'textdomain'),
    'modulebox_gallery_callback',
    'modulebox',
    'normal',
    'high'
  );
}
add_action('add_meta_boxes', 'modulebox_gallery_meta_box');

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

function save_modulebox_gallery($post_id)
{
  if (!isset($_POST['modulebox_gallery_nonce_field']) || !wp_verify_nonce($_POST['modulebox_gallery_nonce_field'], 'modulebox_gallery_nonce')) return;
  if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) return;
  if (!current_user_can('edit_post', $post_id)) return;

  update_post_meta($post_id, '_modulebox_gallery', isset($_POST['modulebox_gallery']) ? array_map('intval', $_POST['modulebox_gallery']) : []);
}
add_action('save_post', 'save_modulebox_gallery');

// 3️⃣  add front-page.php 

// 4️⃣ AJAX Handler for Content Update
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

  ob_start();
?>
<h2><?php echo esc_html($post->post_title); ?></h2>
<?php echo apply_filters('the_content', $post->post_content); ?>
<?php if (has_post_thumbnail($post_id)) : ?>
<a href="<?php echo esc_url(wp_get_attachment_url(get_post_thumbnail_id($post_id))); ?>" data-lightbox="featured-image">
    <?php echo get_the_post_thumbnail($post_id, 'full'); ?>
</a>
<?php endif; ?>
<a href="#message-us" class="btn btn-dark mt-3" data-title="<?php echo esc_attr($post->post_title); ?>">Go to Message
    Us</a>
<?php

  wp_send_json_success(ob_get_clean());
}
add_action('wp_ajax_load_modulebox', 'load_modulebox_content');
add_action('wp_ajax_nopriv_load_modulebox', 'load_modulebox_content');



and also the ########## modulebox-ajax.js FILE CODE:
jQuery(document).ready(function ($) {
    function attachEventHandlers() {
        // Handle 'Go to Message Us' button click
        $(".go-to-message-us").off("click").on("click", function () {
            var postTitle = $(this).data("title");
            $("#post-title").val(postTitle);
            $("#your-subject").val(postTitle);
        });
    }

    // Attach initial event handlers
    attachEventHandlers();

    // Handle module link click
    $("#modules__url a").click(function (e) {
        e.preventDefault();
        var postID = $(this).data("id");

        $.ajax({
            url: ajax_object.ajaxurl,
            type: "POST",
            data: {
                action: "load_module_content",
                nonce: ajax_object.nonce,
                post_id: postID,
            },
            success: function (response) {
                if (response.success) {
                    // Update the gallery (left col-7)
                    $("#modulesCarousel .carousel-inner").html(response.data.gallery);

                    // Update the content (right col-5)
                    $(".module-details").html(response.data.content);

                    // Reinitialize the carousel
                    $("#modulesCarousel").carousel();

                    // Reinitialize Lightbox
                    lightbox.init();

                    // Reattach event handlers for the new content
                    attachEventHandlers();
                } else {
                    alert("Failed to load content.");
                }
            },
        });
    });
});

Thanks for reading this far! Is there anything you can tell?
