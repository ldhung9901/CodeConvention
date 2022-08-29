# Coding Convention (C#)

## UpperCamelCase Case
- Sử dụng cho đặt tên file.
- Sử dụng cho public medthods và properties.
```csharp
public class DataService
{
    public string Id { get; set; }
    public string Name { get; set; }
    public string Address { get; set; }
}
```


- Sử dụng khi đặt tên `class`, `record`, hoặc `struct`.

```csharp
public class DataService
{
}
```

```csharp
public record PhysicalAddress(
    string street,
    string city,
    string stateOrProvince,
    string zipCode);
```

```csharp
public struct ValueCoordinate
{/Users/techpro-mac/QUAN.LY.SAN.XUAT.MES/techpro.zapp/techpro.zapp/ClientApp/src/app/modules/maintenance/maintenance_schedual_system_device
}
```

Sử dụng khi đặt tên interface, thêm prefix `I` trước name.

```csharp
public interface IWorkerQueue
{
}
```

Sử dụng khi đặt tên cho `public` types, fields, properties, events, methods, và local functions.

```csharp
public class ExampleEvents
{
    // A public field, these should be used sparingly
    public bool IsValid;

    // An init-only property
    public IWorkerQueue WorkerQueue { get; init; }

    // An event
    public event Action EventProcessing;

    // Method
    public void StartEventProcessing()
    {
        // Local function
        static int CountQueueItems() => WorkerQueue.Count;
        // ...
    }
}
```
## UpperCamelCase
Sử dụng cho các scope variable.
```csharp
public async Task<int> Update(SystemHelpModel model)
{
    var name  = model.Db.Name;
    var address  = model.Db.Address;

    return 1;
}
```
### Camel case

Sử dụng khi `private` hoặc `internal` fields, và thêm prefix `_`.

```csharp
public class DataService
{
    private IWorkerQueue _workerQueue;
}
```

Sử dụng với `static` fields có AC là `private` hoặc `internal`, sử dụng prefix `s_` và cho thread static `t_`.

```csharp
public class DataService
{
    private static IWorkerQueue s_workerQueue;

    [ThreadStatic]
    private static TimeSpan t_timeSpan;
}
```

Sử dụng với `arguments` của `method`, sử dụng `lowerCamelCase`.

```csharp
public T SomeMethod<T>(int someNumber, bool isValid)
{
}
```

### Sử dụng Named Arguments trong việc gọi method:

When calling a method, arguments are passed with the parameter name followed by a colon and a value.

```csharp
// Method
public void DoSomething(string foo, int bar)
{
...
}

// Avoid
DoSomething("someString", 1);
// Correct
DoSomething(foo: "someString", bar: 1);
```

## Quy tắc comment

- Sử dụng `///` để mô tả chức năng của field. (VS Code Extension).
- Đối với `method` cần mô tả thêm ý nghĩa có giá trị trả về (nếu có)

```csharp
/// <summary>
/// id unit table
/// </summary>
/// <value></value>
[MaxLength(128)]
public string IdUnit { get; set; }
```

```csharp
/// <summary>
/// Func update record help
/// </summary>
/// <param name="model">SystemHelpModel</param>
/// <returns>1 if success</returns>
public async Task<int> Update(SystemHelpModel model)
{
    var db = await _context.SystemHelps.Where(d => d.Id ==  model.Db.Id).FirstOrDefaultAsync();
    db.Name = model.Db.Name;
    db.UpdateBy = model.Db.UpdateBy;
    db.UpdateDate = model.db.UpdateDate;
    _context.SaveChanges();
    return 1;
}
```

## Quy tắc liên quan EF

- Nên sử dụng `async` `Task` nếu có thể.
- Phần xử lý logic liên quan đến database bắt buộc viết tại Repository Layer.
- Sử dụng `[MaxLength(128)]` đối với column ID.
- Sử dụng `[MaxLength(200)]` đối với column Name.
- Sử dụng `[MaxLength(500)]` đối với column Note.
- Bắt buộc thêm `nullable` đối với column (trừ primary key).

```csharp
public int? Id {get; set;}
public string Name {get; set;}
public string Address {get; set;}
```

- Bắt buộc phải khai báo `type` của `field` khi tạo DB mới

```csharp
[MaxLength(128)]
public string IdUnit { get; set; }
```
## Quy tắc viết Unit Test
- Mỗi `Action` trong `Controller` bắt buộc phải có ít nhất 1 `Unit test`.
- Viết test case đúng chức năng của một `Action` trong `Controller`.
- Không sử dụng `decouple` `Action` trong `Controller` trong một Unit test.
```csharp
public class SystemItemTestController : InitHttpContextController<SystemItemController> 
{

    public SystemItemTestController() : base(new DatabaseFixture())
    {
        controller = new sys_itemController(userService: userService, context: fixture.context, serviceFactory: serviceFactory);
    }
    [Fact]
    public async Task GetItemByBarcodeAsync_ShouldReturn200Status()
    {
        // Arrange
        var result = (JsonResult)await controller.getItemByBarcode("", "");

        //Assert
        result.StatusCode.Should().NotBe(500);
    }
    [Fact]
    public async Task DeleteAsync_ShouldDeletedAndReturn200Status()
    {
        // Arrange
        var newId = Guid.NewGuid().ToString();
        fixture.context.sys_items.Add(new sys_item_db
        {
            Id = newId
        });

        var identity = new GenericIdentity("TestUser", "UnitTest");
        var contextUser = new ClaimsPrincipal(identity); 
        controller.ControllerContext = new ControllerContext
        {
            HttpContext = new DefaultHttpContext { User = contextUser }
        };

        await fixture.context.SaveChangesAsync();
        // Act
        var result = controller.delete(JObject.FromObject(new { Id = newId }));

        // Assert
        var record = await fixture.context.SystemItems.Where(d => d.Id == newId).SingleOrDefaultAsync();
        Assert.Equal(expected :2,actual: record?Istatus_del);
        result.StatusCode.Should().Be(200);
        result.StatusCode.Should().NotBe(500);
    }
}
```
## Quy tắc khác

- Khi `method` bị khai báo quá 3 lần thì nên viết lại tại `Helpers`
- Đặt tên file theo `UpperCamelCase` và có suffix (`Repo`, `Model`, `Part`, `Controller`, `Db`)
- Đặt tên file theo `UpperCamelCase` và có prefix là tên Module

```csharp
SysItemRepo.cs
SysItemModel.cs
SysItemPart.cs
InventoryItemModel.cs
```

- Controller chỉ chức business logic không tương tác với Database
- Controller bắt buộc phải có Http Method và cần mô tả chi tiết prams, chức năng của controller

```csharp
/// <summary>
/// Get list of helps.
/// </summary>
/// <param name="item"></param>
/// <returns>Returns the array of helps</returns>
/// <remarks>
/// Sample request:
///
///     POST /Todo
///     {
///        "Id": 1,
///        "Name": "Item #1",
///        "IsComplete": true
///     }
///
/// </remarks>
/// <response code="200">Returns the array of helps</response>
[HttpPost]
[ProducesResponseType(typeof(IEnumerable<CmSelectStringModel>),StatusCodes.Status200Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public IActionResult GetListUse()
{
    var result = repo._context.SystemHelps
        .Where(d=>d.StatusDel==1)
            .Select(d => new CmSelectStringModel
            {
                Id = d.Id,
                Name = d.Name
            }).ToList();
    return Json(result);
}
```

# Coding Convention (Typescript)

| Style          | Category                                                           |
| :------------- | :----------------------------------------------------------------- |
| UpperCamelCase | class / interface / type / enum / decorator / type parameters      |
| lowerCamelCase | variable / parameter / function / method / property / module alias |
| CONSTANT_CASE  | global constant values, including enum values                      |
| #ident         | private identifiers are never used.                                |

```typescript
const UNIT_SUFFIXES = {
  milliseconds: "ms",
  seconds: "s",
};
```

- Khi so sánh bằng bắt buộc sử dụng `===`

```typescript
a === b; // Good
a == b; // Bad
```

- Bắt buộc sử dụng `arrow function` của ES6

```typescript
bar(() => { this.doSomething(); }) // Good

bar(function() { ... }) // Bad
```

- Sử dụng single quote

```typescript
const foo = "bar"; // Good
const foo = "bar"; // Bad
```

- Chỉ sử dụng {} khi đúng trường hợp

```typescript
myPromise.then((v) => console.log(v)); // Bad

myPromise.then((v) => {
  console.log(v);
}); // Good
```

- Bắt buộc phải khai báo và sử dụng type (trừ những trường hợp đặc biệt sử dụng `any`)
- Đặt tên file mới theo đúng cấu trúc (`lowerCamelCase`)
  >
      .
      ├── ...
      ├── systemItem                  # Tên trang có prefix tên module system
      │   ├── export.ts               # Export chung của tất cả component trong page
      │   ├── index.component.ts      # Component chính
      │   ├── index.component.html    # Giao diện của component chính
      │   ├── popupAdd.component.ts   # Popup component liên quan xem, sửa, xoá
      │   ├── popupAdd.component.html # Giao diện popup component liên quan xem, sửa, xoá
      │   ├── types.ts                # Khai báo type cho toàn bộ page
      │   ...
      │   ├── index.service.ts        # Service để cho các component khác sử dụng
      │   └── index.resolver.ts       # Resolver để lấy dữ liệu trước khi render component
      ├── system.module.ts            # Module system
      ├── system.routing.ts           # Routing Module system
      └── system.helper.ts            # Helper (các hàm sử dụng lại)
- Khi khai báo trong component trong module, sử dụng `import` từ `./export`, không import trực tiếp

```typescript
import { inventory_report_position_map_indexComponent } from "./inventory_report_position_map/export"; // Good
import { inventory_report_position_map_indexComponent } from "./inventory_report_position_map/index.component"; // Bad
```

- Bắt buộc sử dụng `Tailwind CSS`, hạn chế sử dụng css thuần.

## Quy tắc comment

- Mô tả nội dung của `function`, `variable` theo như mẫu

```typescript
/**
 * Get navigation component from the registry
 *
 * @param name
 */
getComponent(name: string) : void
{
    return this._componentRegistry.get(name);
}
```

# Coding Convention (Dart - Flutter)

- Classes, enum types, typedefs, và type parameters thì sử dụng `UpperCamelCase`.

```dart
class SliderMenu { ... }

class HttpRequest { ... }

typedef Predicate<T> = bool Function(T value);
```

- Tên libraries, packages, directories, và source files sử dụng `lowercase_with_underscores`.

```dart
library peg_parser.source_scanner;

import 'file_system.dart';
import 'slider_menu.dart';
```

- Trường hợp còn lại sử dụng `lowerCamelCase`.

```dart
var count = 3;

HttpRequest httpRequest;

void align(bool clearItems) {
  // ...
}
```

- Sử dụng `single quote`

```dart
var foo = 'bar'; // Good
var foo = "bar"; // Bad
```

- Sắp xếp `import` đúng thứ tự

```dart
import 'dart:async';
import 'dart:html';

import 'package:bar/bar.dart';
import 'package:foo/foo.dart';
```

- Comment mô tả chức năng của các `Functions`, `Properties`.

````dart
/// URL avatar.
///
/// ```dart
/// var example = CodeBlockExample();
/// print(example.isItGreat); // "Yes."
/// ```
/// ABC XYZ.
String? avatarPath;
````

## Cấu trúc source

>

      lib
      ├── app
      ├── common
      │   ├── config
      │   ├── constant                                  # Biến dùng chung
      │   ├── model                                     # Model của toàn bộ app
      │   ├── preference                                # Function shared_preference
      │   ├── route                                     # Route cho toàn bộ app
      │   ├── util                                      # Các Function sử dụng lại
      │   └── widget
      ├── feature                                       # Danh sách các tính năng của app
      │   ├── login
      │   │   ├── bloc                                  # Bloc của tính năng (nhiều bloc khác nhau)
      │   │   │   ├── export.dart                       # Export chung của Bloc
      │   │   │   ├── login_bloc.dart                   # Bloc xử lý của login feature
      │   │   │   ├── login_event.dart                  # Event Bloc của login feature
      │   │   │   └── login_state.dart                  # State Bloc của login feature
      │   │   ├── repository
      │   │   │   └── login_repository.dart             # Repository (sử dụng để call API Login)
      │   │   └── screen
      │   │   │   └── login.dart                        # Screen chính login tổng hợp từ các widget
      │   │   └── widget
      │   │       └── button_login.dart                 # Widget của screen login
      │   └── ...
      └── translations                                  # Cấu hình đa ngôn ngữ
