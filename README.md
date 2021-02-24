# Laravel_Handle_Mail_Providers_From_DB
laravel handle mail providers ( config/mail.php ) from database .

Database Table email_providers

```
id	int	
name	varchar
transport	varchar
host	varchar	
port	varchar	
encryption	varchar
from	varchar
username	varchar
password	varchar
auth_mode	varchar
timeout	varchar	
is_default	tinyint	
created_at	timestamp
updated_at	timestamp

```


App\Models\EmailProvider
```
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class EmailProvider extends Model
{
    protected $table = 'email_providers' ;
    protected $guarded = [] ;
}
```

App\Providers\MailServiceProvider
```
<?php

namespace App\Providers;

use App\Models\EmailProvider;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\ServiceProvider;

class MailServiceProvider extends ServiceProvider{

    public function boot()
    {
        $config = Cache::remember(env('DEFAULT_MAIL_CACHE_KEY') ,9999999, function () {

            $providers = EmailProvider::all()->collect() ;
            $defaultProvider = $providers->where('is_default' ,1)->first() ;

            if( $providers && $defaultProvider ) {
                $providersConfig = Config::get('mail.mailers') ;
                foreach ( $providers as $provider )
                {
                    $proverConfig = array(
                        'transport'     => $provider['transport'] ,
                        'host'       => $provider['host'] ,
                        'port'       => $provider['port'] ,
                        'encryption' => $provider['encryption'] ,
                        'username'   => $provider['username'] ,
                        'password'   => $provider['password'] ,
                        'timeout' => $provider['password'] ,
                        'auth_mode' => $provider['password'] ,
                    );
                    $providersConfig[ $provider['name'] ] = $proverConfig ;
                }
                Config::set('mail.mailers',$providersConfig) ;
                Config::set('mail.default', $defaultProvider['name'] );
                Config::set('mail.from', array('address' => $defaultProvider['from'], 'name' => env('APP_NAME')));
            }
            return Config::get('mail');
        });

        Config::set('mail',$config) ;
    }

    public function register()
    {

    }

}


```

register provider in config/app.php
```
    'providers' => [
      \App\Providers\MailServiceProvider::class,
    ]
```

Notification
```
<?php

namespace App\Notifications\User\Account;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;
use Illuminate\Queue\SerializesModels;

class RegisterInfo extends Notification
{
    use Queueable , SerializesModels ;

    public function __construct()
    {
        $this->subject = "Account Information" ;
    }

   //mail
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.user.account.register_info', [
                'user' => $notifiable ,
            ]
        )->subject($this->subject);
    }


}


```
