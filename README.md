Paperclip
=========

## Documentation valid for `master` branch

Please check the documentation for the paperclip version you are using:
https://github.com/thoughtbot/paperclip/releases

---

[![Build Status](https://secure.travis-ci.org/thoughtbot/paperclip.svg?branch=master)](http://travis-ci.org/thoughtbot/paperclip)
[![Dependency Status](https://gemnasium.com/thoughtbot/paperclip.svg?travis)](https://gemnasium.com/thoughtbot/paperclip)
[![Code Climate](https://codeclimate.com/github/thoughtbot/paperclip.svg)](https://codeclimate.com/github/thoughtbot/paperclip)
[![Inline docs](http://inch-ci.org/github/thoughtbot/paperclip.svg)](http://inch-ci.org/github/thoughtbot/paperclip)
[![Security](https://hakiri.io/github/thoughtbot/paperclip/master.svg)](https://hakiri.io/github/thoughtbot/paperclip/master)

- [Requirements](#requirements)
  - [Ruby and Rails](#ruby-and-rails)
  - [Image Processor](#image-processor)
  - [`file`](#file)
- [Installation](#installation)
- [Quick Start](#quick-start)
  - [Models](#models)
  - [Migrations](#migrations)
  - [Edit and New Views](#edit-and-new-views)
  - [Edit and New Views with Simple Form](#edit-and-new-views-with-simple-form)
  - [Controller](#controller)
  - [Show View](#show-view)
  - [Deleting an Attachment](#deleting-an-attachment)
  - [Uploading Multiple Images](#uploading-multiple-images)
- [Usage](#usage)
- [Validations](#validations)
- [Internationalization (I18n)](#internationalization-i18n)
- [Security Validations](#security-validations)
- [Defaults](#defaults)
- [Migrations](#migrations-1)
  - [Add Attachment Column To A Table](#add-attachment-column-to-a-table)
  - [Schema Definition](#schema-definition)
  - [Vintage syntax](#vintage-syntax)
- [Storage](#storage)
  - [Understanding Storage](#understanding-storage)
- [Post Processing](#post-processing)
- [Events](#events)
- [URI Obfuscation](#uri-obfuscation)
- [MD5 Checksum / Fingerprint](#md5-checksum--fingerprint)
- [File Preservation for Soft-Delete](#file-preservation-for-soft-delete)
- [Custom Attachment Processors](#custom-attachment-processors)
- [Dynamic Configuration](#dynamic-configuration)
  - [Dynamic Styles](#dynamic-styles)
  - [Dynamic Processors](#dynamic-processors)
- [Logging](#logging)
- [Deployment](#deployment)
- [Testing](#testing)
- [Contributing](#contributing)
- [License](#license)
- [About thoughtbot](#about-thoughtbot)

Paperclip is intended as an easy file attachment library for ActiveRecord. The
intent behind it was to keep setup as easy as possible and to treat files as
much like other attributes as possible. This means they aren't saved to their
final locations on disk, nor are they deleted if set to nil, until
ActiveRecord::Base#save is called. It manages validations based on size and
presence, if required. It can transform its assigned image into thumbnails if
needed, and the prerequisites are as simple as installing ImageMagick (which,
for most modern Unix-based systems, is as easy as installing the right
packages). Attached files are saved to the filesystem and referenced in the
browser by an easily understandable specification, which has sensible and
useful defaults.

See the documentation for `has_attached_file` in [`Paperclip::ClassMethods`](http://www.rubydoc.info/gems/paperclip/Paperclip/ClassMethods) for
more detailed options.

The complete [RDoc](http://www.rubydoc.info/gems/paperclip) is online.

---

Requirements
------------

### Ruby and Rails

Paperclip now requires Ruby version **>= 2.1** and Rails version **>= 4.2**
(only if you're going to use Paperclip with Ruby on Rails.)

### Image Processor

[ImageMagick](http://www.imagemagick.org) must be installed and Paperclip must have access to it. To ensure
that it does, on your command line, run `which convert` (one of the ImageMagick
utilities). This will give you the path where that utility is installed. For
example, it might return `/usr/local/bin/convert`.

Then, in your environment config file, let Paperclip know to look there by adding that
directory to its path.

In development mode, you might add this line to `config/environments/development.rb)`:

```ruby
Paperclip.options[:command_path] = "/usr/local/bin/"
```

If you're on Mac OS X, you'll want to run the following with Homebrew:

    brew install imagemagick

If you are dealing with pdf uploads or running the test suite, you'll also need
to install GhostScript. On Mac OS X, you can also install that using Homebrew:

    brew install gs

If you are on Ubuntu (or any Debian base Linux distribution), you'll want to run
the following with apt-get:

    sudo apt-get install imagemagick -y

### `file`

The Unix [`file` command](https://en.wikipedia.org/wiki/File_(command)) is required for content-type checking.
This utility isn't available in Windows, but comes bundled with Ruby [Devkit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit),
so Windows users must make sure that the devkit is installed and added to the system `PATH`.

**Manual Installation**

If you're using Windows 7+ as a development environment, you may need to install the `file.exe` application manually. The `file spoofing` system in Paperclip 4+ relies on this; if you don't have it working, you'll receive `Validation failed: Upload file has an extension that does not match its contents.` errors.

To manually install, you should perform the following:

> **Download & install `file` from [this URL](http://gnuwin32.sourceforge.net/packages/file.htm)**

To test, you can use the image below:
![untitled](https://cloud.githubusercontent.com/assets/1104431/4524452/a1f8cce4-4d44-11e4-872e-17adb96f79c9.png)

Next, you need to integrate with your environment - preferably through the `PATH` variable, or by changing your `config/environments/development.rb` file

**PATH**

    1. Click "Start"
    2. On "Computer", right-click and select "Properties"
    3. In Properties, select "Advanced System Settings"
    4. Click the "Environment Variables" button
    5. Locate the "PATH" var - at the end, add the path to your newly installed `file.exe` (typically `C:\Program Files (x86)\GnuWin32\bin`)
    6. Restart any CMD shells you have open & see if it works

OR

**Environment**

    1. Open `config/environments/development.rb`
    2. Add the following line: `Paperclip.options[:command_path] = 'C:\Program Files (x86)\GnuWin32\bin'`
    3. Restart your Rails server

Either of these methods will give your Rails setup access to the `file.exe` functionality, thus providing the ability to check the contents of a file (fixing the spoofing problem)

---

Installation
------------

Paperclip is distributed as a gem, which is how it should be used in your app.

Include the gem in your Gemfile:

```ruby
gem "paperclip", "~> 5.0"
```

Or, if you want to get the latest, you can get master from the main paperclip repository:

```ruby
gem "paperclip", git: "git://github.com/thoughtbot/paperclip.git"
```

If you're trying to use features that don't seem to be in the latest released gem, but are
mentioned in this README, then you probably need to specify the master branch if you want to
use them. This README is probably ahead of the latest released version if you're reading it
on GitHub.

For Non-Rails usage:

```ruby
class ModuleName < ActiveRecord::Base
  include Paperclip::Glue
  ...
end
```

---

Quick Start
-----------

### Models

```ruby
class User < ActiveRecord::Base
  has_attached_file :avatar, styles: { medium: "300x300>", thumb: "100x100>" }, default_url: "/images/:style/missing.png"
  validates_attachment_content_type :avatar, content_type: /\Aimage\/.*\Z/
end
```

### Migrations

```ruby
class AddAvatarColumnsToUsers < ActiveRecord::Migration
  def up
    add_attachment :users, :avatar
  end

  def down
    remove_attachment :users, :avatar
  end
end
```

(Or you can use the Rails migration generator: `rails generate paperclip user avatar`)

### Edit and New Views

```erb
<%= form_for @user, url: users_path, html: { multipart: true } do |form| %>
  <%= form.file_field :avatar %>
<% end %>
```

### Edit and New Views with Simple Form
```erb
<%= simple_form_for @user, url: users_path do |form| %>
  <%= form.input :avatar, as: :file %>
<% end %>
```

### Controller

```ruby
def create
  @user = User.create( user_params )
end

private

# Use strong_parameters for attribute whitelisting
# Be sure to update your create() and update() controller methods.

def user_params
  params.require(:user).permit(:avatar)
end
```

### Show View

```erb
<%= image_tag @user.avatar.url %>
<%= image_tag @user.avatar.url(:medium) %>
<%= image_tag @user.avatar.url(:thumb) %>
```

### Deleting an Attachment

Set the attribute to `nil` and save.

```ruby
@user.avatar = nil
@user.save
```
---
### Uploading Multiple Images

If you need to upload multiple images to create something like a gallery,
you can begin by creating a "A Gallery has many Pictures" relationship, and "Picture has one avatar".
```ruby
class Gallery < ActiveRecord::Base
  has_many :pictures, dependent: :destroy
end

class Picture < ActiveRecord::Base
  has_attached_file :avatar, styles: { medium: "300x300>", thumb: "100x100>" }, default_url: "/images/:style/missing.png"
  validates_attachment_content_type :avatar, content_type: /\Aimage\/.*\Z/
  belongs_to :gallery
end

```

Then you enable multiple option in the form, and use 'file_field_tag' to pass array of avatars.
```erb
<%= f.label :Avatars %>
<%= file_field_tag "avatars[]", type: :file , multiple: 'true'%>
```

Now in the create action in GalleryController, you can use create method on each pictures.
```ruby
def create
  @gallery = Gallery.new(gallery_params)
  if @gallery.save
    if params[:avatars]
      params[:avatars].each do |avatar|
        @gallery.pictures.create(avatar: avatar)
      end
    end
    redirect_to @gallery
  else
    render 'new'
  end
end
```
---

Usage
-----

The basics of Paperclip are quite simple: Declare that your model has an
attachment with the `has_attached_file` method, and give it a name.

Paperclip will wrap up to four attributes (all prefixed with that attachment's name,
so you can have multiple attachments per model if you wish) and give them a
friendly front end. These attributes are:

* `<attachment>_file_name`
* `<attachment>_file_size`
* `<attachment>_content_type`
* `<attachment>_updated_at`

By default, only `<attachment>_file_name` is required for Paperclip to operate.
You'll need to add `<attachment>_content_type` in case you want to use content type
validation.

More information about the options passed to `has_attached_file` is available in the
documentation of [`Paperclip::ClassMethods`](http://www.rubydoc.info/gems/paperclip/Paperclip/ClassMethods).

Validations
-----------

For validations, Paperclip introduces several validators to validate your attachment:

* `AttachmentContentTypeValidator`
* `AttachmentPresenceValidator`
* `AttachmentSizeValidator`

Example Usage:

```ruby
validates :avatar, attachment_presence: true
validates_with AttachmentPresenceValidator, attributes: :avatar
validates_with AttachmentSizeValidator, attributes: :avatar, less_than: 1.megabytes

```

Validators can also be defined using the old helper style:

* `validates_attachment_presence`
* `validates_attachment_content_type`
* `validates_attachment_size`

Example Usage:

```ruby
validates_attachment_presence :avatar
```

Lastly, you can also define multiple validations on a single attachment using `validates_attachment`:

```ruby
validates_attachment :avatar, presence: true,
  content_type: { content_type: "image/jpeg" },
  size: { in: 0..10.kilobytes }
```

_NOTE: Post-processing will not even **start** if the attachment is not valid
according to the validations. Your callbacks and processors will **only** be
called with valid attachments._

```ruby
class Message < ActiveRecord::Base
  has_attached_file :asset, styles: {thumb: "100x100#"}

  before_post_process :skip_for_audio

  def skip_for_audio
    ! %w(audio/ogg application/ogg).include?(asset_content_type)
  end
end
```

If you have other validations that depend on assignment order, the recommended
course of action is to prevent the assignment of the attachment until
afterwards, then assign manually:

```ruby
class Book < ActiveRecord::Base
  has_attached_file :document, styles: {thumbnail: "60x60#"}
  validates_attachment :document, content_type: { content_type: "application/pdf" }
  validates_something_else # Other validations that conflict with Paperclip's
end

class BooksController < ApplicationController
  def create
    @book = Book.new(book_params)
    @book.document = params[:book][:document]
    @book.save
    respond_with @book
  end

  private

  def book_params
    params.require(:book).permit(:title, :author)
  end
end
```

**A note on content_type validations and security**

You should ensure that you validate files to be only those MIME types you
explicitly want to support.  If you don't, you could be open to
<a href="https://www.owasp.org/index.php/Testing_for_Stored_Cross_site_scripting_(OWASP-DV-002)">XSS attacks</a>
if a user uploads a file with a malicious HTML payload.

If you're only interested in images, restrict your allowed content_types to
image-y ones:

```ruby
validates_attachment :avatar,
  content_type: { content_type: ["image/jpeg", "image/gif", "image/png"] }
```

`Paperclip::ContentTypeDetector` will attempt to match a file's extension to an
inferred content_type, regardless of the actual contents of the file.

---

Internationalization (I18n)
---------------------------

For using or adding locale files in different languages, check the project
https://github.com/thoughtbot/paperclip-i18n.

Security Validations
====================

Thanks to a report from [Egor Homakov](http://homakov.blogspot.com/) we have
taken steps to prevent people from spoofing Content-Types and getting data
you weren't expecting onto your server.

NOTE: Starting at version 4.0.0, all attachments are *required* to include a
content_type validation, a file_name validation, or to explicitly state that
they're not going to have either. *Paperclip will raise an error* if you do not
do this.

```ruby
class ActiveRecord::Base
  has_attached_file :avatar
  # Validate content type
  validates_attachment_content_type :avatar, content_type: /\Aimage/
  # Validate filename
  validates_attachment_file_name :avatar, matches: [/png\Z/, /jpe?g\Z/]
  # Explicitly do not validate
  do_not_validate_attachment_file_type :avatar
end
```

This keeps Paperclip secure-by-default, and will prevent people trying to mess
with your filesystem.

NOTE: Also starting at version 4.0.0, Paperclip has another validation that
cannot be turned off. This validation will prevent content type spoofing. That
is, uploading a PHP document (for example) as part of the EXIF tags of a
well-formed JPEG. This check is limited to the media type (the first part of the
MIME type, so, 'text' in `text/plain`). This will prevent HTML documents from
being uploaded as JPEGs, but will not prevent GIFs from being uploaded with a
`.jpg` extension. This validation will only add validation errors to the form. It
will not cause errors to be raised.

This can sometimes cause false validation errors in applications that use custom
file extensions. In these cases you may wish to add your custom extension to the
list of content type mappings by creating `config/initializers/paperclip.rb`:

```ruby
# Allow ".foo" as an extension for files with the MIME type "text/plain".
Paperclip.options[:content_type_mappings] = {
  foo: %w(text/plain)
}
```

---

Defaults
--------
Global defaults for all your Paperclip attachments can be defined by changing the Paperclip::Attachment.default_options Hash. This can be useful for setting your default storage settings per example so you won't have to define them in every `has_attached_file` definition.

If you're using Rails, you can define a Hash with default options in `config/application.rb` or in any of the `config/environments/*.rb` files on config.paperclip_defaults. These will get merged into `Paperclip::Attachment.default_options` as your Rails app boots. An example:

```ruby
module YourApp
  class Application < Rails::Application
    # Other code...

    config.paperclip_defaults = { storage: :fog, fog_credentials: { provider: "Local", local_root: "#{Rails.root}/public"}, fog_directory: "", fog_host: "localhost"}
  end
end
```

Another option is to directly modify the `Paperclip::Attachment.default_options` Hash - this method works for non-Rails applications or is an option if you prefer to place the Paperclip default settings in an initializer.

An example Rails initializer would look something like this:

```ruby
Paperclip::Attachment.default_options[:storage] = :fog
Paperclip::Attachment.default_options[:fog_credentials] = { provider: "Local", local_root: "#{Rails.root}/public"}
Paperclip::Attachment.default_options[:fog_directory] = ""
Paperclip::Attachment.default_options[:fog_host] = "http://localhost:3000"
```
---

Migrations
----------

Paperclip defines several migration methods which can be used to create the necessary columns in your
model. There are two types of helper methods to aid in this, as follows:

### Add Attachment Column To A Table

The `attachment` helper can be used when creating a table:

```ruby
class CreateUsersWithAttachments < ActiveRecord::Migration
  def up
    create_table :users do |t|
      t.attachment :avatar
    end
  end

  # This is assuming you are only using the users table for Paperclip attachment. Drop with care!
  def down
    drop_table :users
  end
end
```

You can also use the `change` method, instead of the `up`/`down` combination above, as shown below:

```ruby
class CreateUsersWithAttachments < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.attachment :avatar
    end
  end
end
```

### Schema Definition

Alternatively, the `add_attachment` and `remove_attachment` methods can be used to add new Paperclip columns to an existing table:

```ruby
class AddAttachmentColumnsToUsers < ActiveRecord::Migration
  def up
    add_attachment :users, :avatar
  end

  def down
    remove_attachment :users, :avatar
  end
end
```

Or you can do this with the `change` method:

```ruby
class AddAttachmentColumnsToUsers < ActiveRecord::Migration
  def change
    add_attachment :users, :avatar
  end
end
```

### Vintage syntax

Vintage syntax (such as `t.has_attached_file` and `drop_attached_file`) is still supported in
Paperclip 3.x, but you're advised to update those migration files to use this new syntax.

---

Storage
-------

Paperclip ships with 3 storage adapters:

* File Storage
* S3 Storage (via `aws-sdk`)
* Fog Storage

If you would like to use Paperclip with another storage, you can install these
gems along side with Paperclip:

* [paperclip-azure-storage](https://github.com/gmontard/paperclip-azure-storage)
* [paperclip-dropbox](https://github.com/janko-m/paperclip-dropbox)

### Understanding Storage

The files that are assigned as attachments are, by default, placed in the
directory specified by the `:path` option to `has_attached_file`. By default, this
location is `:rails_root/public/system/:class/:attachment/:id_partition/:style/:filename`.
This location was chosen because, on standard Capistrano deployments, the
`public/system` directory is symlinked to the app's shared directory, meaning it
will survive between deployments. For example, using that `:path`, you may have a
file at

    /data/myapp/releases/20081229172410/public/system/users/avatar/000/000/013/small/my_pic.png

_**NOTE**: This is a change from previous versions of Paperclip, but is overall a
safer choice for the default file store._

You may also choose to store your files using Amazon's S3 service. To do so, include
the `aws-sdk` gem in your Gemfile:

```ruby
gem 'aws-sdk', '>= 2.0.34'
```

And then you can specify using S3 from `has_attached_file`.
You can find more information about configuring and using S3 storage in
[the `Paperclip::Storage::S3` documentation](http://www.rubydoc.info/gems/paperclip/Paperclip/Storage/S3).

Files on the local filesystem (and in the Rails app's public directory) will be
available to the internet at large. If you require access control, it's
possible to place your files in a different location. You will need to change
both the `:path` and `:url` options in order to make sure the files are unavailable
to the public. Both `:path` and `:url` allow the same set of interpolated
variables.

---

Post Processing
---------------

Paperclip supports an extensible selection of post-processors. When you define
a set of styles for an attachment, by default it is expected that those
"styles" are actually "thumbnails." However, you can do much more than just
thumbnail images. By defining a subclass of Paperclip::Processor, you can
perform any processing you want on the files that are attached. Any file in
your Rails app's `lib/paperclip` and `lib/paperclip_processors` directories is
automatically loaded by Paperclip, allowing you to easily define custom
processors. You can specify a processor with the `:processors` option to
`has_attached_file`:

```ruby
has_attached_file :scan, styles: { text: { quality: :better } },
                         processors: [:ocr]
```

This would load the hypothetical class Paperclip::Ocr, which would have the
hash "{ quality: :better }" passed to it along with the uploaded file. For
more information about defining processors, see Paperclip::Processor.

The default processor is Paperclip::Thumbnail. For backward compatibility
reasons, you can pass a single geometry string or an array containing a
geometry and a format that the file will be converted to, like so:

```ruby
has_attached_file :avatar, styles: { thumb: ["32x32#", :png] }
```

This will convert the "thumb" style to a 32x32 square in PNG format, regardless
of what was uploaded. If the format is not specified, it is kept the same (i.e.
JPGs will remain JPGs). For more information on the accepted style formats, see
[here](http://www.imagemagick.org/script/command-line-processing.php#geometry).

Multiple processors can be specified, and they will be invoked in the order
they are defined in the `:processors` array. Each successive processor will
be given the result of the previous processor's execution. All processors will
receive the same parameters, which are defined in the `:styles` hash.
For example, assuming we had this definition:

```ruby
has_attached_file :scan, styles: { text: { quality: :better } },
                         processors: [:rotator, :ocr]
```

then both the :rotator processor and the :ocr processor would receive the
options `{ quality: :better }`. This parameter may not mean anything to one
or more or the processors, and they are expected to ignore it.

_NOTE: Because processors operate by turning the original attachment into the
styles, no processors will be run if there are no styles defined._

If you're interested in caching your thumbnail's width, height and size in the
database, take a look at the [paperclip-meta](https://github.com/teeparham/paperclip-meta) gem.

Also, if you're interested in generating the thumbnail on-the-fly, you might want
to look into the [attachment_on_the_fly](https://github.com/drpentode/Attachment-on-the-Fly) gem.

---

Events
------

Before and after the Post Processing step, Paperclip calls back to the model
with a few callbacks, allowing the model to change or cancel the processing
step. The callbacks are `before_post_process` and `after_post_process` (which
are called before and after the processing of each attachment), and the
attachment-specific `before_<attachment>_post_process` and
`after_<attachment>_post_process`. The callbacks are intended to be as close to
normal ActiveRecord callbacks as possible, so if you return false (specifically
\- returning nil is not the same) in a `before_filter`, the post processing step
will halt. Returning false in an `after_filter` will not halt anything, but you
can access the model and the attachment if necessary.

_NOTE: Post processing will not even **start** if the attachment is not valid
according to the validations. Your callbacks and processors will **only** be
called with valid attachments._

```ruby
class Message < ActiveRecord::Base
  has_attached_file :asset, styles: {thumb: "100x100#"}

  before_post_process :skip_for_audio

  def skip_for_audio
    ! %w(audio/ogg application/ogg).include?(asset_content_type)
  end
end
```

---

URI Obfuscation
---------------

Paperclip has an interpolation called `:hash` for obfuscating filenames of
publicly-available files.

Example Usage:

```ruby
has_attached_file :avatar, {
    url: "/system/:hash.:extension",
    hash_secret: "longSecretString"
}
```


The `:hash` interpolation will be replaced with a unique hash made up of whatever
is specified in `:hash_data`. The default value for `:hash_data` is `":class/:attachment/:id/:style/:updated_at"`.

`:hash_secret` is required - an exception will be raised if `:hash` is used without `:hash_secret` present.

For more on this feature, read [the author's own explanation](https://github.com/thoughtbot/paperclip/pull/416)

MD5 Checksum / Fingerprint
-------

An MD5 checksum of the original file assigned will be placed in the model if it
has an attribute named fingerprint.  Following the user model migration example
above, the migration would look like the following:

```ruby
class AddAvatarFingerprintColumnToUser < ActiveRecord::Migration
  def up
    add_column :users, :avatar_fingerprint, :string
  end

  def down
    remove_column :users, :avatar_fingerprint
  end
end
```

File Preservation for Soft-Delete
-------

An option is available to preserve attachments in order to play nicely with soft-deleted models. (acts_as_paranoid, paranoia, etc.)

```ruby
has_attached_file :some_attachment, {
    preserve_files: "true",
}
```

This will prevent ```some_attachment``` from being wiped out when the model gets destroyed, so it will still exist when the object is restored later.

---

Custom Attachment Processors
-------

Custom attachment processors can be implemented and their only requirement is
to inherit from `Paperclip::Processor` (see `lib/paperclip/processor.rb`).
For example, when `:styles` are specified for an image attachment, the
thumbnail processor (see `lib/paperclip/thumbnail.rb`) is loaded without having
to specify it as a `:processor` parameter to `has_attached_file`.  When any
other processor is defined, it must be called out in the `:processors`
parameter if it is to be applied to the attachment.  The thumbnail processor
uses the ImageMagick `convert` command to do the work of resizing image
thumbnails.  It would be easy to create a custom processor that watermarks
an image using ImageMagick's `composite` command.  Following the
implementation pattern of the thumbnail processor would be a way to implement a
watermark processor.  All kinds of attachment processors can be created;
a few utility examples would be compression and encryption processors.

---

Dynamic Configuration
---------------------

Callable objects (lambdas, Procs) can be used in a number of places for dynamic
configuration throughout Paperclip.  This strategy exists in a number of
components of the library but is most significant in the possibilities for
allowing custom styles and processors to be applied for specific model
instances, rather than applying defined styles and processors across all
instances.

### Dynamic Styles:

Imagine a user model that had different styles based on the role of the user.
Perhaps some users are bosses (e.g. a User model instance responds to `#boss?`)
and merit a bigger avatar thumbnail than regular users. The configuration to
determine what style parameters are to be used based on the user role might
look as follows where a boss will receive a `300x300` thumbnail otherwise a
`100x100` thumbnail will be created.

```ruby
class User < ActiveRecord::Base
  has_attached_file :avatar, styles: lambda { |attachment| { thumb: (attachment.instance.boss? ? "300x300>" : "100x100>") } }
end
```

### Dynamic Processors:

Another contrived example is a user model that is aware of which file processors
should be applied to it (beyond the implied `thumbnail` processor invoked when
`:styles` are defined). Perhaps we have a watermark processor available and it is
only used on the avatars of certain models.  The configuration for this might be
where the instance is queried for which processors should be applied to it.
Presumably some users might return `[:thumbnail, :watermark]` for its
processors, where a defined `watermark` processor is invoked after the
`thumbnail` processor already defined by Paperclip.

```ruby
class User < ActiveRecord::Base
  has_attached_file :avatar, processors: lambda { |instance| instance.processors }
  attr_accessor :processors
end
```

---

Logging
----------

By default, Paperclip outputs logging according to your logger level. If you want to disable logging (e.g. during testing) add this into your environment's configuration:
```ruby
Your::Application.configure do
...
  Paperclip.options[:log] = false
...
end
```

More information in the [rdocs](http://www.rubydoc.info/github/thoughtbot/paperclip/Paperclip.options)

---

Deployment
----------

Paperclip is aware of new attachment styles you have added in previous deploys. The only thing you should do after each deployment is to call
`rake paperclip:refresh:missing_styles`.  It will store current attachment styles in `RAILS_ROOT/public/system/paperclip_attachments.yml`
by default. You can change it by:

```ruby
Paperclip.registered_attachments_styles_path = '/tmp/config/paperclip_attachments.yml'
```

Here is an example for Capistrano:

```ruby
namespace :deploy do
  desc "build missing paperclip styles"
  task :build_missing_paperclip_styles do
    on roles(:app) do
      within release_path do
        with rails_env: fetch(:rails_env) do
          execute :rake, "paperclip:refresh:missing_styles"
        end
      end
    end
  end
end

after("deploy:compile_assets", "deploy:build_missing_paperclip_styles")
```

Now you don't have to remember to refresh thumbnails in production every time you add a new style.
Unfortunately, it does not work with dynamic styles - it just ignores them.

If you already have a working app and don't want `rake paperclip:refresh:missing_styles` to refresh old pictures, you need to tell
Paperclip about existing styles. Simply create a `paperclip_attachments.yml` file by hand. For example:

```ruby
class User < ActiveRecord::Base
  has_attached_file :avatar, styles: { thumb: 'x100', croppable: '600x600>', big: '1000x1000>' }
end

class Book < ActiveRecord::Base
  has_attached_file :cover, styles: { small: 'x100', large: '1000x1000>' }
  has_attached_file :sample, styles: { thumb: 'x100' }
end
```

Then in `RAILS_ROOT/public/system/paperclip_attachments.yml`:

```yml
---
:User:
  :avatar:
  - :thumb
  - :croppable
  - :big
:Book:
  :cover:
  - :small
  - :large
  :sample:
  - :thumb
```

---

Testing
-------

Paperclip provides rspec-compatible matchers for testing attachments. See the
documentation on [Paperclip::Shoulda::Matchers](http://www.rubydoc.info/gems/paperclip/Paperclip/Shoulda/Matchers)
for more information.

**Parallel Tests**

Because of the default `path` for Paperclip storage, if you try to run tests in
parallel, you may find that files get overwritten because the same path is being
calculated for them in each test process. While this fix works for
parallel_tests, a similar concept should be used for any other mechanism for
running tests concurrently.

```ruby
if ENV['PARALLEL_TEST_GROUPS']
  Paperclip::Attachment.default_options[:path] = ":rails_root/public/system/:rails_env/#{ENV['TEST_ENV_NUMBER'].to_i}/:class/:attachment/:id_partition/:filename"
else
  Paperclip::Attachment.default_options[:path] = ":rails_root/public/system/:rails_env/:class/:attachment/:id_partition/:filename"
end
```

The important part here being the inclusion of `ENV['TEST_ENV_NUMBER']`, or a
similar mechanism for whichever parallel testing library you use.

**Integration Tests**

Using integration tests with FactoryGirl may save multiple copies of
your test files within the app. To avoid this, specify a custom path in
the `config/environments/test.rb` like so:

```ruby
Paperclip::Attachment.default_options[:path] = "#{Rails.root}/spec/test_files/:class/:id_partition/:style.:extension"
```

Then, make sure to delete that directory after the test suite runs by adding
this to `spec_helper.rb`.

```ruby
config.after(:suite) do
  FileUtils.rm_rf(Dir["#{Rails.root}/spec/test_files/"])
end
```
---

Contributing
------------

If you'd like to contribute a feature or bugfix: Thanks! To make sure your
fix/feature has a high chance of being included, please read the following
guidelines:

1. Post a [pull request](https://github.com/thoughtbot/paperclip/compare/).
2. Make sure there are tests! We will not accept any patch that is not tested.
   It's a rare time when explicit tests aren't needed. If you have questions
   about writing tests for paperclip, please open a
   [GitHub issue](https://github.com/thoughtbot/paperclip/issues/new).

Please see `CONTRIBUTING.md` for more details on contributing and running test.

Thank you to all [the contributors](https://github.com/thoughtbot/paperclip/graphs/contributors)!

License
-------

Paperclip is Copyright © 2008-2016 thoughtbot, inc. It is free software, and may be
redistributed under the terms specified in the MIT-LICENSE file.

About thoughtbot
----------------

![thoughtbot](https://thoughtbot.com/logo.png)

Paperclip is maintained and funded by thoughtbot.
The names and logos for thoughtbot are trademarks of thoughtbot, inc.

We love open source software!
See [our other projects][community] or
[hire us][hire] to design, develop, and grow your product.

[community]: https://thoughtbot.com/community?utm_source=github
[hire]: https://thoughtbot.com?utm_source=github
