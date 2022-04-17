# Create and update migrations for auto deployment

# Challenge (چالش)

جداول یک سرویس بعد از تحلیل اولیه و ساخت سرویس توسط فایل‌های migration ایجاد و ساخته می‌شوند.
مادامی که سرویس دست خوش تغییر می‌شود جداول هم تغییر خواهند کرد. برای استقرار توسعه‌ها (امکانات) در محیط‌های مختلف از develop تا production  نیاز داریم تغییرات دیتابیس را در کد(migrations) داشته باشیم و به راحتی آن را اعمال کنیم.
چالش اصلی این است، اگر برای هر تغییر کوچکی یک فایل migration ایجاد کنیم بعداز مدتی تعداد فایل‌ها بسیار می‌شود و تمیز دادن آنها دشوار و احتمال خطا را افزایش می‌دهد.

**راه‌حل:**
هر جدول یک فایل اصلی migration داشته باشد این فایل همان فایلی است که در ابتدای ایجاد سرویس و یا پروژه لاراول ایجاد می‌شود. با تغییر پروژه زمانی که تغییرات جدول انجام می‌شود این فایل هم بروزرسانی می‌شود.
هر زمانی قصد کنیم سرویس را از ابتدا مستقر (deploy) کنیم این فایل به روز است و به سهولت عملیات استقرار انجام می‌شود.
بخش اصلی چالش، اعمال تغییرات جدید دیتابیس بروی محیط‌های است که سرویس در آن‌ها از قبل مستقر شده است.
برای اعمال تغییرات جدید جداول، می‌توان آنها را در فایل‌های جدیدی به صورت موقت ایجاد کرد این تغییرات به واسطه cicd در محیط‌های فعال اعمال خواهد شد. مسیر فایل‌های موقت زیر پوشه updates  در پوشه migrations تعیین شده است.
تغییرات توسط برنامه‌نویس در این فایل‌ها و در مسیر تعیین شده ایجاد می‌شود و زمانی که امکانات جدید مستقر می‌شود این فایل‌ها هم فراخوانی و اعمال خواهند شد.
فایل‌های ایجاد شده از زمان ایجاد تا اعمال و تست بروی محیط develop و همچنین اعمال آن بروی محیط production برای استفاده کاربر نهایی، نگه‌داری خواهند شد و بعد از اطمینان از این‌که تغییرات بروی تمام محیط‌ها اعمال شده است می‌توان آنها را حذف کرد.

**نکات:**
گاها ممکن است تغییراتی را از فایل‌های موقت خوانده و اعمال کنیم که قبلا توسط فایل اصلی اعمال شده‌اند. این رویداد باعث خطا شده و روند استقرار CICD را با خطا مواجه و ادامه آن را دچار اختلال می‌کند.
همچنین در هنگام استقرار یک سرویس و یا توسعه سرویس ممکن است خطای رخ دهد و نیاز باشد تا عملیات استقرار مجددا انجام شود. 
مثال: ممکن است برنامه‌نویس یک امکانی (feature) را توسعه داده و این امکانات با تغییرات دیتابیس هم‌راه باشد، برنامه‌نویس علاوه بر اینکه فایل اصلی ایجاد جدول را بروز کرده است فایل‌های موقتی را هم در پوشه updates قرار داده است. 
در این هنگام نیاز می‌شود سرویس مذکور در محیط جدید مستقر شود، به قطع می‌توان گفت عملیات استقرار با مشکل مواجه خواهد شد.

**پیش نیاز:** 
برای اعمال تغییراتی مانند تعویض نوع ستون و …. نیاز است پکیج doctrine/dbal را نصب کنید.

**برای مدیریت اتفاقات فوق نیاز است در هنگام تعریف فایل‌های migration مواردی را رعایت کنیم.**
**این موارد در ادامه این سند شرح داده شده است.**

# CREATION (ساخت جدول)
    زمانی که یک جدول ساخته می‌شود ساختار اولیه آن درفایلی با الگوی زیر در مسیر database/migrations قرار می‌گیرد.
    این فایل در تمام مدتی که ساختار دیتابیس تغییر می‌کند بروز می‌شود.
    بهتر است تغییرات ساختاری دیتابیس (مانند ساخت، حذف و یا ویرایش یک ستون) به صورت migration انجام شود، و هیچ تغییری به صورت دستی انجام نشود.
    در زمان استقرار (deploy) سرویس از دستور php artisan migrate استفاده می‌شود.
    این دستور با هر بار ارسال تغییرات به ریپو سرویس اعمال خواهد شد.

Migration path:

    database/migrations/

Migration file pattern:

    YYYY-mm-dd_(int{6})_create_table_name_table.php

Create migration (code sample)

    php artisan make:migration create_table_name

Best Practice (99%): Create table code sample, without error in DevOps (CICD):

    public function up()
    {
      if (Schema::hasTable('table_name')){
        return ;
      }
    
      Schema::create('table_name', static function (Blueprint $table) {
        $table->bigIncremenAts('id');
        ....
      }
    }
# UPDATE (تغییر ستون‌های جدول)
    زمانی که می‌خواهیم یک ستون به جدولی اضافه یا حذف و یا ویرایش کنیم.
    ابتدا تغییرات را در فایل اصلی migration اعمال می‌کنیم.
    سپس یک فایل ‌migration با نام و الگوی زیر در مسیر database/migrations/updates  ایجاد می‌کنیم تغییرات را با alter در فایل ایجاد شده اعمال می‌کنیم و به شاخه مورد نظر ریپو ارسال (push) می‌کنیم.
    این تغییرات با هر بار ارسال فایل‌ها به ریپو سرویس اعمال خواهد شد.

Updates migrations path

    database/migrations/updates

Alter migration file pattern

    add_{column_name}_column_to_{table_name}.php

Create migration file (code sample)

    php artisan make:migrate add_foloan_column_to_my_table --path="database/migrations/updates"

Best Practice (99%): Edit column code sample, without error in DevOps (CICD):
in up

    public function up()
    {
      Schema::table($this->table, function (Blueprint $table) {
        //add column if not exists in table
        if (! Schema::hasColumn('table_name', 'column_name')){
            $table->string('column_name');
        }
      
        //change column type and nulllable
        if (Schema::hasColumn('table_name', 'column_name')){
            $table->string('column_name', 50)->nullable()->change();
        }
      
        //rename column
        if (Schema::hasColumn('table_name', 'column_name')){
            $table->renameColumn('column_name', 'to');
        }
        //drop column
        if (Schema::hasColumn('table_name', 'column_name')){
            $table->dropColumn('column_name');
        }
      }
    }

in down

    public function down()
    {
      //drop table
      Schema::dropIfExists('table_name');
    
      //drop column
      if (Schema::hasColumn('table_name', 'column_name')){
          $table->dropColumn('column_name');
      }
    }

and more items in laravel doc:
https://laravel.com/docs/7.x/migrations


# رعایت نکات
1. نوع  ستون داخلی با ستون خارجی باید یکسان باشد.
2. ستون داخلی باید no null باشد.
3. 


