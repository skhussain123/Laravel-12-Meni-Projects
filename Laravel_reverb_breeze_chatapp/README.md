```bash
composer create-project laravel/laravel lara_breeze_chatapp-app

composer require laravel/breeze --dev
php artisan breeze:install blade

composer require livewire/livewire

npm install && npm run dev
php artisan install:broadcasting -->reverb  -->yes  -->  yes  

php artisan reverb:start --debug
```

* Seeder/DatabaseSeeder.php
```bash
 User::factory()->create([
            'name' => 'Test User 1',
            'email' => 'test1@example.com',
            'password' => 1234,
        ]);


        User::factory()->create([
            'name' => 'Test User 1',
            'email' => 'test2@example.com',
             'password' => 1234,
        ]);

        User::factory()->create([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
             'password' => 1234,
        ]);
``` 

```bash
php artisan make:model Message -m
```
```bash
$table->foreignId('sender_id');
$table->foreignId('receiver_id');
$table->text('message');
``` 

```bash
php artisan migrate
php artisan migrate:fresh --seed
```

#### Message Model 
```bash
  public function sender()
    {
        return $this->belongsTo(User::class, 'sender_id');
    }

    public function receiver()
    {
        return $this->belongsTo(User::class, 'receiver_id');
    }
```

#### web.php
```bash
Route::get('/dashboard', [ProfileController::class, 'dashboard'])->middleware(['auth', 'verified'])->name('dashboard');
```

### ProfileController
```bash
  public function dashboard()
    {
        $currentUser = Auth::user();
        $users = User::where('id', '!=', $currentUser->id)->get();

        return view('dashboard', compact('users'));
    }
```  

#### dashboard.blade.php
```bash
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Dashboard') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
    

                    @foreach ($users as $user)
                        <a href="{{ route('chatdetail', ['id' => $user->id]) }}">
                            {{$user->name}}
                            {{$user->email}}

                            <br>
                        </a>
                    @endforeach

                </div>
            </div>
        </div>
    </div>
</x-app-layout>

```

#### Add Routes web.php
```bash
Route::get('/chat/{id}', [ProfileController::class, 'chatDetail'])->name('chatdetail');
```


#### Add ProfileController
```bash
 public function chatDetail($id)
    {
        return view('chat-detail', compact('id'));
    }

```

#### views/chat-detail.blade.php
```bash
<x-app-layout>
    @livewire('chat-component', ['user_id' => $id])
</x-app-layout>
```


#### Create Chat-Component
```bash
php artisan make:livewire ChatComponent
```

#### LiveWire/chat-component.blade.php
```bash
<div>
    <script type="module" src="https://cdn.jsdelivr.net/npm/emoji-picker-element@^1/index.js"></script>

    <style>
        body {
            margin: 0;
            font-family: Arial, sans-serif;
            overscroll-behavior: none;
            background-color: #f5f5f5;
        }

        .header {
            position: fixed;
            top: 0;
            width: 100%;
            background-color: #38a169;
            color: white;
            height: 64px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 12px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
            z-index: 10;
        }

        .chat-box {
            margin-top: 80px;
            margin-bottom: 80px;
            padding: 10px;
        }

        .message {
            padding: 10px 15px;
            margin: 10px;
            border-radius: 10px;
            display: inline-block;
            max-width: 75%;
            word-wrap: break-word;
            clear: both;
        }

        .message.left {
            background-color: #d1d5db;
            float: left;
            text-align: left;
        }

        .message.right {
            background-color: #86efac;
            float: right;
            text-align: right;
        }

        .footer {
            position: fixed;
            bottom: 0;
            width: 100%;
            background-color: #bbf7d0;
            display: flex;
            align-items: center;
            padding: 8px;
            box-shadow: 0 -2px 4px rgba(0, 0, 0, 0.1);
        }

        .textarea {
            flex-grow: 1;
            border: 1px solid #ccc;
            background-color: #e5e7eb;
            border-radius: 9999px;
            padding: 10px 15px;
            resize: none;
            outline: none;
            margin-right: 8px;
        }

        .send-btn {
            background: none;
            border: none;
            padding: 0;
            outline: none;
            cursor: pointer;
        }

        .icon {
            width: 32px;
            height: 32px;
        }

        .username {
            font-weight: bold;
            font-size: 18px;
        }

        .icon-dots,
        .icon-back {
            width: 24px;
            height: 24px;
        }
    </style>


    <!-- Header -->
    <div class="header">
        <!-- Back Button -->
        <a href="{{ route('dashboard') }}">
            <svg class="icon-back" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="#d1fae5">
                <path
                    d="M9.41 11H17a1 1 0 0 1 0 2H9.41l2.3 2.3a1 1 0 1 1-1.42 1.4l-4-4a1 1 0 0 1 0-1.4l4-4a1 1 0 0 1 1.42 1.4L9.4 11z" />
            </svg>
        </a>

        <div class="username">{{ $user->name }}<br>{{ $user->email }}</div>


        <!-- 3 Dots -->
        <svg class="icon-dots" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="#d1fae5">
            <path fill-rule="evenodd"
                d="M12 7a2 2 0 1 1 0-4 2 2 0 0 1 0 4zm0 7a2 2 0 1 1 0-4 2 2 0 0 1 0 4zm0 7a2 2 0 1 1 0-4 2 2 0 0 1 0 4z" />
        </svg>
    </div>

    <!-- Chat Messages -->
    <div class="chat-box">
        @foreach ($messages as $message)
            @php
                $isFile = Str::startsWith($message['message'], 'chat_files/');
                $fileExtension = pathinfo($message['message'], PATHINFO_EXTENSION);
            @endphp

            @if ($message['sender'] != auth()->user()->name)
                <div class="message left">
                    {{ $message['sender'] }} :
                    @if ($isFile)
                        <div class="flex items-center gap-2 text-sm text-gray-800">
                            📎 {{ basename($message['message']) }}
                            <a href="{{ asset('storage/' . $message['message']) }}" download
                                class="text-blue-600 underline ml-2">
                                Download
                            </a>
                        </div>
                    @else
                        {{ $message['message'] }}
                    @endif
                    <p style="font-size: 12px">{{ $message['created_at'] }}</p>
                </div>
            @else
                <div class="message right">
                    @if ($isFile)
                        <div class="flex items-center gap-2 text-sm text-gray-800">
                            📎 {{ basename($message['message']) }}
                            <a href="{{ asset('storage/' . $message['message']) }}" download
                                class="text-blue-600 underline ml-2">
                                Download
                            </a>
                        </div>
                    @else
                        {{ $message['message'] }}
                    @endif


                    <br>
                    <p style="font-size: 12px">{{ $message['created_at'] }}</p>
                </div>
            @endif

            <br><br><br><br>
        @endforeach



        <!-- 👇 File preview will show here -->
        <div class="show-file px-4 py-2">
            @if ($file)
                <div class="flex justify-center">
                    <div class="bg-green-100 border border-green-400 rounded-lg p-3 max-w-xs w-fit shadow-md relative">
                        <!-- File Name -->
                        <div class="text-sm text-green-800 truncate">
                            📎 {{ $file->getClientOriginalName() }}
                        </div>

                        <!-- Remove Button -->
                        <button wire:click="removeFile" type="button"
                            class="absolute top-1 right-1 text-red-600 font-bold text-sm">
                            &times;
                        </button>

                    </div>
                </div>
            @endif
        </div>


    </div>


    <!-- Footer -->
    <form wire:submit.prevent="sendMessage" enctype="multipart/form-data">
        <div class="footer">
            @csrf

            <!-- Emoji Button -->
            <button type="button" id="emoji-toggle" class="send-btn">😊</button>

            <!-- Emoji Picker -->
            <emoji-picker id="emoji-picker"
                style="position: absolute; bottom: 60px; left: 10px; display: none; z-index: 999;"></emoji-picker>

            <!-- 📎 File Input -->
            <label class="send-btn" style="margin-right: 8px;">
                📎
                <input type="file" wire:model="file" style="display: none;">
            </label>

            <!-- 📝 Textarea -->
            <textarea class="textarea" wire:model.defer="message" rows="1" placeholder="Message..." id="message-textarea"
                @if ($file) disabled @endif></textarea>



            <!-- 📤 Send Button -->
            <button class="send-btn" type="submit">
                <svg class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" fill="#38a169">
                    <path
                        d="M476 3.2L12.5 270.6c-18.1 10.4-15.8 35.6 2.2 43.2L121 358.4l287.3-253.2c5.5-4.9 13.3 2.6 8.6 8.3L176 407v80.5c0 23.6 28.5 32.9 42.5 15.8L282 426l124.6 52.2c14.2 6 30.4-2.9 33-18.2l72-432C515 7.8 493.3-6.8 476 3.2z" />
                </svg>
            </button>
        </div>
    </form>


    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const picker = document.getElementById('emoji-picker');
            const toggle = document.getElementById('emoji-toggle');
            const textarea = document.getElementById('message-textarea');

            // Toggle emoji picker
            toggle.addEventListener('click', () => {
                picker.style.display = picker.style.display === 'none' ? 'block' : 'none';
            });

            // Insert emoji into textarea
            picker.addEventListener('emoji-click', event => {
                const emoji = event.detail.unicode;
                const start = textarea.selectionStart;
                const end = textarea.selectionEnd;
                const text = textarea.value;
                textarea.value = text.substring(0, start) + emoji + text.substring(end);
                textarea.focus();
                textarea.setSelectionRange(start + emoji.length, start + emoji.length);

                // Manually dispatch input event so Livewire detects change
                textarea.dispatchEvent(new Event('input'));
            });
        });
    </script>


</div>
```

### Create Event For BroatCasting
```bash
php artisan make:event MessageSentEvent
```

#### Event/MessageSentEvent

```bash
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class MessageSentEvent implements ShouldBroadcastNow
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $message;

    /**
     * Create a new event instance.
     */
    public function __construct($message)
    {
        $this->message = $message;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('chat-channel.' . $this->message->receiver_id),
        ];
    }
}

```

#### routes/channels.php
```bash

use Illuminate\Support\Facades\Broadcast;
use App\Models\User;


Broadcast::channel('App.Models.User.{id}', function ($user, $id) {
    return (int) $user->id === (int) $id;
});

Broadcast::channel('chat-channel.{userId}',function(User $user, $userId){
    return (int) $user->id === (int) $userId;
});

```


#### LiveWire/ChatComponent
```bash
<?php

namespace App\Livewire;

use App\Models\Message;
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Events\MessageSentEvent;
use Livewire\Attributes\On;
use Livewire\Features\SupportFileUploads\WithFileUploads;

class ChatComponent extends Component
{
    use WithFileUploads;

    public $user;
    public $sender_id;
    public $receiver_id;
    public $message;
    public $messages = [];
    public $file;



    public function render()
    {
        return view('livewire.chat-component');
    }

    public function mount($user_id)
    {
        $this->sender_id = Auth::user()->id;
        $this->receiver_id = $user_id;

        $messages = Message::where(function ($query) {
            $query->where('sender_id', $this->sender_id)
                ->where('receiver_id', $this->receiver_id);
        })
            ->orWhere(function ($query) {
                $query->where('sender_id', $this->receiver_id)
                    ->where('receiver_id', $this->sender_id);
            })
            ->with('sender:id,name', 'receiver:id,name')
            ->get();

        foreach ($messages as $message) {
            $this->appendChatMessage($message);
        }

        $this->user = User::findOrFail($user_id);
    }

    public function removeFile()
    {
        $this->file = null;
    }


    public function sendMessage()
    {
        $chatMessage = new Message();
        $chatMessage->sender_id = $this->sender_id;
        $chatMessage->receiver_id = $this->receiver_id;

        if ($this->file) {
            // File selected: Upload file and store its path in message column
            $filename = $this->file->store('chat_files', 'public');
            $chatMessage->message = $filename;

            $this->file = null;
        } else {
            // No file selected: Save plain text message
            $chatMessage->message = $this->message;
            $this->message = "";
        }

        $chatMessage->save();

        $this->appendChatMessage($chatMessage);

        broadcast(new MessageSentEvent($chatMessage))->toOthers();
    }


    #[On('echo-private:chat-channel.{sender_id},MessageSentEvent')]
    public function listenForMessage($event)
    {
        $chatMessage = Message::whereId($event['message']['id'])
            ->with('sender:id,name', 'receiver:id,name')
            ->first();

        $this->appendChatMessage($chatMessage);
    }


    public function appendChatMessage($message)
    {
        $this->messages[] = [
            'id' => $message->id,
            'message' => $message->message,
            'sender' => $message->sender->name,
            'receiver' => $message->receiver->name,
            'created_at' => $message->created_at->format('h:i A')

        ];
    }
}

```

```bash
php artisan storage:link
```


#### Make Sure Krna ha ye Command Run krke project test krna hain
```bash
php artisan serve
npm run dev
php artisan reverb:start --debug
```












