
##### install 
composer require google/apiclient


#### goto the googe cloud console and create project

1. go to library and enabled Google Drive api
2. Google Drive API page top me create Credential pr click krke select application data and next
3.  fill service account information
4. Click oauth content screen fill Project configuration
**--> App name**
**--> External**

5. go to Credentials and create Aouth Client id secret id
**---> web application**
**--->use redirect url https://developers.google.com/oauthplayground**
**----> set app name**
6. copy auth client id and secret id
7. goto Audience and application Publishing status  publich app


8. goto url https://developers.google.com/oauthplayground settings icon click check the auth credentials
and find Drive Api select and check related link and send request and genrate to copy Refresh token:


9. Gogo Google Drive Create Folder and copy folder id

10. Set config/services.php
```bash
'google' => [
        'client_id' => env('GOOGLE_CLIENT_ID'),
        'client_secret' => env('GOOGLE_CLIENT_SECRET'),
        'refresh_token' => env('GOOGLE_REFRESH_TOKEN'),
        'folder_id' => env('GOOGLE_FOLDER_ID'),
    ],
```    

11--.env 
```bash
GOOGLE_CLIENT_ID=41667477313-472kn6bb2t56ogp1q9aikxxxxxxxxxxxxxxxxxxxxxx
GOOGLE_CLIENT_SECRET=GOCSPX-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GOOGLE_REFRESH_TOKEN=1//041LZgLHXKIxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GOOGLE_FOLDER_ID=1JyIrqZ2xxxxxxxxxxxxxxxxxxx
```



### Welcome.blade.php
```bash
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet"
        integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    <title>File Upload</title>
</head>

<body>
    <div class="container">
        <div class="card ">
            <div class="card-header">
                Upload Files
            </div>
            <div class="card-body">
                <form action="{{ route('storeFile') }}" method="POST" enctype="multipart/form-data">
                    @csrf
                    <div class="mb-3">
                        <label for="exampleInputEmail1" class="form-label">Name</label>
                        <input name="file_name" type="text" class="form-control" id="exampleInputEmail1"
                            aria-describedby="emailHelp" required>
                    </div>
                    <div class="mb-3">
                        <label for="exampleInputPassword1" class="form-label">File Upload</label>
                        <input name="file" type="file" class="form-control" id="exampleInputPassword1" required>
                    </div>
                    <button name="file" type="submit" class="btn btn-success">Submit</button>
                </form>

                <h1>File downloads</h1>
                <table class="table">
                    <thead>
                        <tr>
                            <th scope="col">#</th>
                            <th scope="col">Name</th>
                            <th scope="col">Action</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach ($files as $file)
                            <tr>
                                <th scope="row">{{ $loop->index + 1 }}</th>
                                <td>{{ $file->file_name }}</td>
                                <td>
                                    <a href="{{ route('download', ['id' => $file->fileid]) }}"
                                        class="btn btn-primary btn-sm">Download</a>

                                </td>
                            </tr>
                        @endforeach
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</body>

</html>

```

### Routes.blade.php
```bash
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\AdminController;

Route::get('/', [AdminController::class, 'create'])->name('index');
Route::post('/file', [AdminController::class, 'productImagesUpload'])->name('storeFile');
Route::get('/download/{id}', [AdminController::class, 'downloadFile'])->name('download');

```

### AdminContoller.php
* php artisan make:Controller AdminController
```bash
<?php

namespace App\Http\Controllers;

use App\Models\File;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

class AdminController extends Controller
{
    public function token()
    {
        $client_id = \Config('services.google.client_id');
        $client_secret = \Config('services.google.client_secret');
        $refresh_token = \Config('services.google.refresh_token');
        $folder_id = \Config('services.google.folder_id');

        $response = Http::post('https://oauth2.googleapis.com/token', [

            'client_id' => $client_id,
            'client_secret' => $client_secret,
            'refresh_token' => $refresh_token,
            'grant_type' => 'refresh_token',

        ]);
        //dd($response);
        $accessToken = json_decode((string) $response->getBody(), true)['access_token'];

        return $accessToken;
    }

    //fetch all data
    public function create()
    {
        $files = File::all();

        return view('welcome', compact('files'));
    }

    // Store data
    public function productImagesUpload(Request $request)
    {
        $validation = $request->validate([
            'file' => 'file|required',
            'file_name' => 'required',
        ]);

        $accessToken = $this->token();
        $name = $request->file->getClientOriginalName();
        $path = $request->file->getRealPath();
        $folderId = \Config('services.google.folder_id');

        // Prepare metadata
        $metadata = [
            'name' => $name,
            'parents' => [$folderId],
        ];

        $response = Http::withHeaders([
            'Authorization' => 'Bearer ' . $accessToken,
        ])
            ->attach(
                'metadata',
                json_encode($metadata),
                'metadata.json'
            )
            ->attach(
                'file',
                file_get_contents($path),
                $name
            )
            ->post('https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart');

        if ($response->successful()) {
            $file_id = json_decode($response->body())->id;

            $uploadedfile = new File();
            $uploadedfile->file_name = $request->file_name;
            $uploadedfile->name = $name;
            $uploadedfile->fileid = $file_id;
            $uploadedfile->save();

            return response('File Uploaded to Google Drive Folder');
        }

        return response('Failed to Upload to Google Drive');
    }


    // Store File Example Livewire Component
    <?php

namespace App\Livewire\Admin;

use App\Models\Category;
use Livewire\Component;
use Livewire\WithFileUploads;
use Illuminate\Support\Facades\Http;

class AddCategory extends Component
{
    use WithFileUploads;

    public $name;
    public $file;

    public function token()
    {
        $client_id = config('services.google.client_id');
        $client_secret = config('services.google.client_secret');
        $refresh_token = config('services.google.refresh_token');

        $response = Http::post('https://oauth2.googleapis.com/token', [
            'client_id' => $client_id,
            'client_secret' => $client_secret,
            'refresh_token' => $refresh_token,
            'grant_type' => 'refresh_token',
        ]);

        return json_decode($response->body(), true)['access_token'];
    }

    public function save()
    {
        $this->validate([
            'name' => 'required|string|max:255',
            'file' => 'required|file|max:2048',
        ]);

        try {
            $accessToken = $this->token();
            $folderId = config('services.google.folder_id');

            $uploadedFileIds = [];

            $fileName = $this->file->getClientOriginalName();
            $filePath = $this->file->getRealPath();

            $metadata = [
                'name' => $fileName,
                'parents' => [$folderId],
            ];

            $response = Http::withHeaders([
                'Authorization' => 'Bearer ' . $accessToken,
            ])
                ->attach('metadata', json_encode($metadata), 'metadata.json')
                ->attach('file', file_get_contents($filePath), $fileName)
                ->post('https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart');

            if ($response->successful()) {
                $file_id = json_decode($response->body())->id;
                $uploadedFileIds[] = $file_id;

                // Make file public
                Http::withToken($accessToken)
                    ->post("https://www.googleapis.com/drive/v3/files/{$file_id}/permissions", [
                        'role' => 'reader',
                        'type' => 'anyone',
                    ]);

    
                // Save to database
                $category = new Category();
                $category->category_name = $this->name;
                $category->category_image = $fileName;
                $category->category_fileid = $file_id;
                $category->status = 'Active';
                $category->save();
            }

            $this->reset(['name', 'file']);
            session()->flash('success', 'File uploaded and category saved successfully.');
        } catch (\Exception $e) {
            session()->flash('error', 'Something went wrong: ' . $e->getMessage());
        }
    }

    public function render()
    {
        return view('livewire.admin.add-category');
    }
}

    //Download file
    public function downloadFile($id)
    {
        $accessToken = $this->token();

        $response = Http::withHeaders([
            'Authorization' => 'Bearer ' . $accessToken,
        ])
            ->get("https://www.googleapis.com/drive/v3/files/{$id}?alt=media");

        if ($response->successful()) {
            // Get file record from DB (optional, if you want the name)
            $file = \App\Models\File::where('fileid', $id)->first();

            return response($response->body())
                ->header('Content-Type', $response->header('Content-Type'))
                ->header('Content-Disposition', 'attachment; filename="' . ($file->name ?? 'file') . '"');
        }

        return response('Download failed', 404);
    }


public function DeleteFile($id, $proid, $fileid)
    {
        try {
            $indexid = Crypt::decrypt($id);
            $proid = Crypt::decrypt($proid);
            $fileid = Crypt::decrypt($fileid);

            // 1. DELETE FILE from Google Drive
            $accessToken = $this->token();
            $deleteResponse = Http::withToken($accessToken)
                ->delete("https://www.googleapis.com/drive/v3/files/{$fileid}");

            if (!$deleteResponse->successful()) {
                return redirect()->back()->with('error', 'Failed to delete file from Google Drive.');
            }

            // 2. REMOVE fileid and its thumbnail from the DB
            $product = Products::findOrFail($proid);
            $fileIds = json_decode($product->product_pic, true);
            $thumbnails = json_decode($product->Thumbnail, true);

            // Remove item at $indexid
            unset($fileIds[$indexid]);
            unset($thumbnails[$indexid]);

            // Reindex array to fix JSON structure
            $product->product_pic = json_encode(array_values($fileIds));
            $product->Thumbnail = json_encode(array_values($thumbnails));
            $product->save();

            return redirect()->back()->with('success', 'File deleted successfully.');
        } catch (Exception $e) {
            return redirect()->back()->with('error', $e->getMessage());
        }
    }
           
}

// update
public function updateCategory(Request $request)
{
    $request->validate([
        'categoryName' => 'required|string|max:255',
        'file_id' => 'required', // Old file_id
        'category_id' => 'required',
    ]);

    $category = Category::findOrFail($request->category_id);

    $accessToken = $this->token(); // Assuming you have this method
    $folderId = config('services.google.folder_id');

    if ($request->hasFile('file')) {
        $file = $request->file('file');
        $fileName = $file->getClientOriginalName();
        $filePath = $file->getRealPath();

        // Upload to Google Drive
        $metadata = [
            'name' => $fileName,
            'parents' => [$folderId],
        ];

        $response = Http::withHeaders([
            'Authorization' => 'Bearer ' . $accessToken,
        ])
            ->attach('metadata', json_encode($metadata), 'metadata.json')
            ->attach('file', file_get_contents($filePath), $fileName)
            ->post('https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart');

        if ($response->successful()) {
            $newFileId = json_decode($response->body())->id;

            // Make file public
            Http::withToken($accessToken)
                ->post("https://www.googleapis.com/drive/v3/files/{$newFileId}/permissions", [
                    'role' => 'reader',
                    'type' => 'anyone',
                ]);

            // Get thumbnail
            $metaResponse = Http::withToken($accessToken)
                ->get("https://www.googleapis.com/drive/v3/files/{$newFileId}", [
                    'fields' => 'thumbnailLink',
                ]);

            $thumbnail = $metaResponse->json()['thumbnailLink'] ?? null;

            // Optional: Delete old file
            $oldFileId = $request->file_id;
            if (!empty($oldFileId)) {
                Http::withToken($accessToken)
                    ->delete("https://www.googleapis.com/drive/v3/files/{$oldFileId}");
            }

            // Update database
            $category->category_name = $request->categoryName;
            $category->category_fileid = $newFileId;
            $category->category_image = $fileName;
            $category->category_thumbnail = $thumbnail;
            $category->save();

            return response()->json([
                'message' => 'Category updated successfully',
                'file_id' => $newFileId,
                'thumbnail' => $thumbnail,
            ]);
        } else {
            return response()->json([
                'message' => 'File upload failed',
                'error' => $response->body(),
            ], 500);
        }
    }

    // If no file was uploaded, just update name
    $category->category_name = $request->categoryName;
    $category->save();

    return response()->json(['message' => 'Category updated (no image change)']);
}






```

### File Migration
* php artisan make:migration Create_Files
```bash
  Schema::create('files', function (Blueprint $table) {
            $table->id();
            $table->string('file_name');
            $table->string('fileid');
            $table->string('name');
            $table->timestamps();
        });
```

### Make File Model
* php artisan make:Model File
```bash
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class File extends Model
{
    //
}

```



