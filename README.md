# visualCaptcha Laravel jQuery

A visual and cleaner alternative to the traditional text based Captcha

This is a Laravel version using jQuery of the Captcha alternative from [http://visualcaptcha.net](http://visualcaptcha.net "http://visualcaptcha.net")
VisualCaptcha is the easiest to implement secure Captcha with images instead of text (and mobile-friendly).


- A better alternative to text based Captchas. 
- Decrease user frustration and improve conversion rates.
- Server side validation.
- Easy to style and add custom images.
- Easily translated text.
- Can be used in Views and Controllers.


![visualCaptcha Laravel jQuery](http://www.hugocabral.pt/visualCaptcha-Laravel-jQuery.png)

## Pre-Requisites
- Laravel
- > PHP 5.3

## Installation Back-End

- Begin by installing this package through Composer.
```js
{
    "require": {
		"hugocabral/visualcaptcha-laravel-jquery": "~1.0"
	}
}
```
- Add these lines top the "app.php" file in the config folder where is the "aliases" category :
```php 
	// app.php
	'Captcha'          => 'visualCaptcha\Captcha',
	'SessionCaptcha'   => 'visualCaptcha\SessionCaptcha',
```

- Update composer in terminal: 
```bash
	composer update
```


### Note
 - Example/ folder contains the app/ and public/ files used for the laravel example shown here: [http://www.hugocabral.pt/visualCaptcha-Laravel-jQuery](http://www.hugocabral.pt/visualCaptcha-Laravel-jQuery "http://www.hugocabral.pt/visualCaptcha-Laravel-jQuery")
 - 
 

## Usage
Note: This implementation uses Localization from Laravel (Languages). No need to freak out. Its easy to use and you can check the explanation in this readme under Localization.

##### Add routes:

```php
// routes.php

		// Check Locale selected and show demo view. If your app will be English only, you can remove: $lang = Config::get('app.locale'); and remove: ,compact('lang').
		Route::get('/', array('as' => 'demo', function()
		{
		    $lang = Config::get('app.locale');
		    return View::make('demo', compact('lang'));
		}));
		
		// Send method to validate captcha
		Route::post('captchacheck','CaptchaController@send');
		
		// Routes for app.visualcaptcha.js. 
		Route::group(array('prefix' => 'captcha'), function()
		{
		    Route::get('start/{howmany}', array('as' => 'captcha/start', 'uses'=>'CaptchaController@start'));
		    Route::get('audio/{type?}', array('as' => 'captcha/audio', 'uses'=>'CaptchaController@audio'));
		    Route::get('image/{index?}', array('as' => 'captcha/image', 'uses'=>'CaptchaController@image'));
		});
```

##### Create CaptchaController.php to create captcha, refresh, validade and do action (ex send email, update database)

```php
// app/controllers/CaptchaController.php

		class CaptchaController extends BaseController {
		
		    /**
		     * Generate a Captcha
		     * @return Response
		     */
		    public function start($howmany)
		    {
		        $session = new SessionCaptcha();
		        $captcha = new Captcha($session);
		        $captcha->generate($howmany);
		
		        return Response::json($captcha->getFrontEndData());
		    }
		
		    /**
		     * Get an audio file
		     * @param  string $type Song type (mp3/ogg)
		     * @return File
		     */
		    public function audio($type = 'mp3')
		    {
		        $session = new SessionCaptcha();
		        $captcha = new Captcha($session);

		        return $captcha->streamAudio(array(), $type);
		    }
		
		    /**
		     * Get an image file
		     * @param  string $index Image index
		     * @return File
		     */
		    public function image($index)
		    {
		        $session = new SessionCaptcha();
		        $captcha = new Captcha($session);

		        return $captcha->streamImage(array(), $index, 0 );
		    }
		
		    /**
		     * Check if the Captcha result is good
		     * @return Mixed
		     */
		    public function send()
		    {
		
		        $data = Input::all();
		
		        //Validation rules
		        $rules = array(
		
		            'name' => 'required'
		        );
		
		        //Validate data
		        $validator = Validator::make($data, $rules);
		
		        //If everything is correct than run passes.
		        if ($validator->passes())
		        {
		            $session = new SessionCaptcha();
		            $captcha = new Captcha($session);
		
		            $frontendData = $captcha->getFrontendData();
		
		            if (!$frontendData)
		            {
		                return Redirect::back()->withInput()->withErrors(array('message' => Lang::get('text.captcha.none')));
		    
		            } else
		            {
		                // If an image field name was submitted, try to validate it
		                if ($imageAnswer = Input::get($frontendData['imageFieldName']))
		                {
		                    if ($captcha->validateImage($imageAnswer))
		                    {
		                        // Return true if the result is correct
		
		                        (... DO ACTION HERE, ex send email, update db...)
		
		                        // This return back to form to show sucess, thats why i use status=1 
		                        return Redirect::back()->with('status', 1);
		
		                    } else
		                    {
		                        return Redirect::back()->withInput()->withErrors(array('message' => Lang::get('text.captcha.image')));
		                        
		                    }
		                } else if ($audioAnswer = Input::get($frontendData['audioFieldName']))
		                {
		                    if ($captcha->validateAudio($audioAnswer))
		                    {
		                        // Return true if the result is correct
		
		                        (... DO ACTION HERE, ex send email, update db...)
		
		                         // This return back to form to show sucess, thats why i use status=1 
		                        return Redirect::back()->with('status', 1);
		
		                    } else
		                    {
		                        return Redirect::back()->withInput()->withErrors(array('message' => Lang::get('text.captcha.audio')));
		                        
		                    }
		                } else
		                {
		                    return Redirect::back()->withInput()->withErrors(array('message' => Lang::get('text.captcha.incomplete')));
		                    
		                }
		            }
		
		        } else
		        {
		            //return "contact form with errors";
		            return Redirect::back()->withErrors($validator)->with('status', 2);;
		
		        }
		
		    }
		}

```
Note: The Lang::get('xxx.xxx.xxx') code is the implementation of Localization (Languages). If your app will be English only, you can hardcore and replace this with your messages/errors. 


##### Include jQuery library, jQuery visualCaptcha front-end library and app.js in you view or main layout or wherever you will be using visualCaptcha
```html
// contact.blade.php
{{ HTML::script(js/jquery.min.js) }}
{{ HTML::script(js/visualcaptcha.jquery.js) }}
{{ HTML::script(js/app.js') }}) }}
```



## README INCOMPLETE / will finish soon



