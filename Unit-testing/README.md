

Laravel unit testing ko asan aur simple tareeqay se samjhtay hain. Testing ka matlab hota hai apnay code ko check karna ke sab kuch sahi kaam kar raha hai ya nahi — automatically.



Unit Test ka matlab hai code ke chhote hisson (functions, classes, models) ko check karna. Ye chhoti chhoti cheezein test karta hai, jaise:

1. ek function sahi result de raha hai ya nahi
2. koi model ka data theek kaam kar raha hai ya nahi

###
| Step | Kaam                                    |
| ---- | --------------------------------------- |
| 1️⃣  | `php artisan make:test TestName --unit` |
| 2️⃣  | Test file me function banao             |
| 3️⃣  | `assertEquals()` se result check karo   |
| 4️⃣  | `php artisan test` se test run karo     |

