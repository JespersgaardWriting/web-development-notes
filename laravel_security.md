Laravel Security
========================


Private Data
-------------

I found a way to make laravel work with private data (eg, images, attachments, etc.). This will be used for documents that should only be seen by authorized users (with potentially different access for each authorized user). Basically, we have to make sure that we get the data through laravel, rather than being served up directly by apache. To do this:

Use this directory structure:

    /project/app/...     (all app code)
            /public/...  (all public assets)
            /private/... (all private assets)
    
We have to look for a file that is not available the file system (if the file exists, Apache will just return it, based on the standard .htaccess rules in Laravel). If the file does not exist, laravel will be called to return something.

In app/routes.php:

    Route::get('/assets/{id1}', 
        array('uses'=>'RoutingController@findAsset'));
    Route::get('/assets/{id1}/{id2}', 
        array('uses'=>'RoutingController@findAsset'));
    Route::get('/assets/{id1}/{id2}/{id3}', 
        array('uses'=>'RoutingController@findAsset'));
    Route::get('/assets/{id1}/{id2}/{id3}/{id4}', 
        array('uses'=>'RoutingController@findAsset'));
    Route::get('/assets/{id1}/{id2}/{id3}/{id4}/{id5}', 
        array('uses'=>'RoutingController@findAsset'));
    Route::get('/assets/{id1}/{id2}/{id3}/{id4}/{id5}/{id6}', 
        array('uses'=>'RoutingController@findAsset'));
    Route::get('/assets/{id1}/{id2}/{id3}/{id4}/{id5}/{id6}/{id7}', 
        array('uses'=>'RoutingController@findAsset'));
    Route::get('/assets/{id1}/{id2}/{id3}/{id4}/{id5}/{id6}/{id7}/{id8}', 
        array('uses'=>'RoutingController@findAsset'));
    Route::get('/assets/{id1}/{id2}/{id3}/{id4}/{id5}/{id6}/{id7}/{id8}/{id9}', 
        array('uses'=>'RoutingController@findAsset'));

The RoutingController looks like this:

    class RoutingController extends Controller
    {
        public function findAsset($p1='', $p2='', $p3='', $p4='', 
            $p5='', $p6='', $p7='', $p8='', $p9='')
        {
            // Find out if the user has permission to use this asset;
            // abort with a 403 error code if they do not.
            // For instance, abort if the user is a guest and does not have a p3 element
            // if (Auth::guest() && strlen($p3)==0)
            //     App::abort(403);

            $asset = $this->joinPath(
                $_SERVER["DOCUMENT_ROOT"] . '/../private',
                $p1, $p2, $p3, $p4, $p5, $p6, $p7, $p8, $p9);

            // Find out if the user has permission to use this asset;
            // abort with a 403 error code if they do not.
    
            $this->returnFile($asset);
        }

        private function joinPath($p0, $p1, $p2, $p3, $p4, $p5, $p6, $p7, $p8, $p9)
        {
            $asset = $p0;

            if (strlen($p1)>0) $asset.='/'.$p1;
            if (strlen($p2)>0) $asset.='/'.$p2;
            if (strlen($p3)>0) $asset.='/'.$p3;
            if (strlen($p4)>0) $asset.='/'.$p4;
            if (strlen($p5)>0) $asset.='/'.$p5;
            if (strlen($p6)>0) $asset.='/'.$p6;
            if (strlen($p7)>0) $asset.='/'.$p7;
            if (strlen($p8)>0) $asset.='/'.$p8;
            if (strlen($p9)>0) $asset.='/'.$p9;

            return realpath($asset);
        }

        private function returnFile($path, array $headers = array())
        {
            if (!is_file($path))
                App::abort(404);

            $finfo = finfo_open(FILEINFO_MIME_TYPE); 
            $mime = finfo_file($finfo, $path);

            // Prepare the headers
            $headers = array_merge(array(
                'Content-Description'       => 'File Transfer',
                'Content-Type'              => $mime,
                'Content-Transfer-Encoding' => 'binary',
                'Expires'                   => 0,
                'Cache-Control'             => 'must-revalidate, post-check=0, pre-check=0',
                'Pragma'                    => 'public',
                'Content-Length'            => filesize($path),
            ), $headers);

            $response = Response::make('', 200, $headers);

            // If there's a session we should save it now
            if (Config::get('session.driver') !== '')
            {
                Session::save();
            }

            // Below is from http://uk1.php.net/manual/en/function.fpassthru.php comments
            session_write_close();
            ob_end_clean();
            $response->sendHeaders();
            if ($file = fopen($path, 'rb')) {
                while(!feof($file) and (connection_status()==0)) {
                    print(fread($file, 1024*8));
                    flush();
                }
                fclose($file);
            }

            // Finish off, like Laravel would
            Event::fire('laravel.done', array($response));
            $response->foundation->finish();

            exit;
        }
    }

So, I'll get these results:

    /                                 (returns homepage)
    /assets/css/style.css             (returns the standard css file)
    /assets/attach/acro.jpg           (returns an error--from RoutingController)
    /assets/attach/2012/acro.jpg      (returns the acro.jpg file--allowed by RoutingController)
    /login                            (returns login page, lets me log in)
    /assets/attach/acro.jpg           (returns the acro.jpg file)
    /assets/attach/2012/acro.jpg      (returns file private/attach/2012/acro.jpg)
    