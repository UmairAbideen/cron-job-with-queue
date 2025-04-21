## 📬 Laravel 10 Cron Job Email with Queue Example
This Laravel 10 project demonstrates how to send emails every minute using a cron job and Laravel Queues. The email logic is moved into a Job class for better performance, allowing asynchronous email delivery without slowing down your application.

## ⏰ Why Use a Cron Job?
Cron jobs automate tasks on a fixed schedule — like sending emails every minute, generating reports, or cleaning data. When paired with Laravel Queues, these jobs can run in the background, so users don’t experience any delay in your app.

## 🔄 Why Combine Cron + Queue?
Cron ensures a task is run regularly (like every minute).
Queue makes sure the job (like sending an email) runs asynchronously, in the background.
Perfect for tasks that take time and shouldn't affect the user experience.

## 🛠️ Tech Stack

Tool	Purpose
Laravel 10	PHP Framework
Mail::send	Send emails (no Mailable used)
Database Queue	Store queued jobs
Blade View	Email content layout

## 📝 Step-by-Step Guide

1. Update .env for queue & mail

```bash

QUEUE_CONNECTION=database

MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your_email@gmail.com
MAIL_PASSWORD=your_app_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=your_email@gmail.com
MAIL_FROM_NAME="Your App"
```

2. Create Job Class
```bash
php artisan make:job SendEmailJob

📁 App\Jobs\SendEmailJob.php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Support\Facades\Mail;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;

class SendEmailJob implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $data;

    public function __construct($data)
    {
        $this->data = $data;
    }

    public function handle(): void
    {
        Mail::send('message', [
            'title' => $this->data['title'],
            'body' => $this->data['body'],
        ], function ($message) {
            $message->to($this->data['email'])
                    ->subject($this->data['subject']);
        });
    }
}

```


3. Create Artisan Command
```bash

php artisan make:command SendEmailCommand
```

📁 App\Console\Commands\SendEmailCommand.php
```bash
 
namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Jobs\SendEmailJob;

class SendEmailCommand extends Command
{
    protected $signature = 'send:email';
    protected $description = 'Send email every minute via cron job';

    public function handle()
    {
        $data = [
            'subject' => 'Email via Cron + Queue',
            'title' => 'Scheduled Email',
            'body' => 'This email is sent via a queued job.',
            'email' => 'you@example.com',
        ];

        dispatch(new SendEmailJob($data));

        $this->info('Email Job Dispatched!');
    }
}

4. Create Blade View
```bash
📁 resources/views/message.blade.php

<h1>{{ $title }}</h1>
<p>{{ $body }}</p>

5. Register Command in Scheduler
```bash
📁 App\Console\Kernel.php

protected function schedule(Schedule $schedule)
{
    $schedule->command('send:email')->everyMinute();
}
```

6. Set Up Queue Table

```bash
php artisan queue:table
php artisan migrate
```

7. Start Queue Worker
```bash

php artisan queue:work
```
## 💡 Keep this running in a separate terminal so jobs can be processed as they’re dispatched.

## 🔧 Test It Manually
```bash
php artisan send:email       # Dispatch job manually
php artisan schedule:run     # Simulate cron for local testing
