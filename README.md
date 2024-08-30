# SSC-Internship-Dev-ASP.NET-Dependency-Injection
# Hướng Dẫn Sử Dụng Dependency Injection Trong ASP.NET Core
## Giới Thiệu
Dependency Injection (DI) là một kỹ thuật phổ biến trong lập trình hướng đối tượng giúp giảm sự phụ thuộc giữa các lớp và tăng tính tái sử dụng cũng như khả năng kiểm thử của mã nguồn. ASP.NET Core tích hợp sẵn DI và cung cấp ba vòng đời dịch vụ chính: Scoped, Transient, và Singleton. Bài viết này sẽ hướng dẫn bạn cách sử dụng DI trong ASP.NET Core, giải thích các vòng đời dịch vụ, và cung cấp ví dụ cụ thể để minh họa.

## 1. Dependency Injection Là Gì?
Dependency Injection là một mẫu thiết kế mà trong đó một lớp (class) không tự tạo ra các phụ thuộc của nó mà các phụ thuộc đó được truyền vào từ bên ngoài. Điều này giúp mã nguồn trở nên dễ bảo trì, dễ mở rộng và dễ kiểm thử.

Ví dụ, thay vì một lớp tự tạo một đối tượng của lớp khác để sử dụng, đối tượng đó sẽ được "tiêm" vào thông qua constructor hoặc các phương thức khác.

## 2. Các Vòng Đời Dịch Vụ (Service Lifetimes)
Trong ASP.NET Core, bạn có thể đăng ký các dịch vụ với ba vòng đời khác nhau:

- Transient: Dịch vụ được tạo mới mỗi khi nó được yêu cầu. Dùng khi bạn muốn mỗi lần sử dụng là một đối tượng khác nhau.
- Scoped: Dịch vụ được tạo một lần duy nhất cho mỗi yêu cầu (HTTP request). Dùng khi bạn muốn sử dụng cùng một đối tượng trong suốt một yêu cầu.
- Singleton: Dịch vụ được tạo một lần duy nhất và được sử dụng lại trong toàn bộ vòng đời của ứng dụng. Dùng khi bạn muốn chia sẻ trạng thái hoặc tài nguyên giữa các yêu cầu.
## 3. Cấu Hình Dependency Injection Trong ASP.NET Core
Trong ASP.NET Core, bạn có thể cấu hình DI bằng cách đăng ký các dịch vụ trong `Program.cs` hoặc `Startup.cs`. Ví dụ sau minh họa cách đăng ký các dịch vụ với các vòng đời khác nhau:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Transient
builder.Services.AddTransient<ITransientService, TransientService>();

// Scoped
builder.Services.AddScoped<IScopedService, ScopedService>();

// Singleton
builder.Services.AddSingleton<ISingletonService, SingletonService>();

var app = builder.Build();

app.Run();
```
Trong ví dụ trên, `ITransientService`, `IScopedService`, và `ISingletonService` là các interface, và `TransientService`, `ScopedService`, `SingletonService` là các lớp triển khai tương ứng.

## 4. Ví Dụ Cụ Thể
Giả sử bạn có ba dịch vụ với các vòng đời khác nhau như sau:

```csharp
public interface ITransientService
{
    Guid GetOperationId();
}

public interface IScopedService
{
    Guid GetOperationId();
}

public interface ISingletonService
{
    Guid GetOperationId();
}

public class OperationService : ITransientService, IScopedService, ISingletonService
{
    private readonly Guid _operationId;

    public OperationService()
    {
        _operationId = Guid.NewGuid();
    }

    public Guid GetOperationId() => _operationId;
}
```
Trong ví dụ này, `OperationService` được triển khai cho cả ba interface với một thuộc tính `_operationId` là `Guid` được tạo mới trong constructor. Mỗi vòng đời dịch vụ sẽ ảnh hưởng đến cách `_operationId` được tạo ra và duy trì.

Tiếp theo, bạn có thể đăng ký các dịch vụ này trong `Program.cs` như sau:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddTransient<ITransientService, OperationService>();
builder.Services.AddScoped<IScopedService, OperationService>();
builder.Services.AddSingleton<ISingletonService, OperationService>();

var app = builder.Build();

app.MapGet("/", (ITransientService transient, IScopedService scoped, ISingletonService singleton) =>
{
    return new
    {
        Transient = transient.GetOperationId(),
        Scoped = scoped.GetOperationId(),
        Singleton = singleton.GetOperationId()
    };
});

app.Run();
```
Ở đây, khi bạn truy cập endpoint `/`, bạn sẽ thấy các `Guid` khác nhau được trả về tùy theo vòng đời dịch vụ:

- Transient: Mỗi lần yêu cầu sẽ tạo một `Guid` mới.
- Scoped: Mỗi lần yêu cầu HTTP sẽ sử dụng cùng một `Guid`.
- Singleton: Toàn bộ ứng dụng chỉ có một `Guid`, được sử dụng lại trong tất cả các yêu cầu.
## 5. Dependency Injection Và Controllers
Bạn có thể dễ dàng sử dụng DI trong các controllers. ASP.NET Core tự động tiêm các dịch vụ vào constructor của controller:

```csharp
[ApiController]
[Route("[controller]")]
public class OperationController : ControllerBase
{
    private readonly ITransientService _transientService;
    private readonly IScopedService _scopedService;
    private readonly ISingletonService _singletonService;

    public OperationController(ITransientService transientService, IScopedService scopedService, ISingletonService singletonService)
    {
        _transientService = transientService;
        _scopedService = scopedService;
        _singletonService = singletonService;
    }

    [HttpGet]
    public IActionResult GetOperations()
    {
        return Ok(new
        {
            Transient = _transientService.GetOperationId(),
            Scoped = _scopedService.GetOperationId(),
            Singleton = _singletonService.GetOperationId()
        });
    }
}
```
Mỗi lần bạn gọi endpoint này, kết quả sẽ phản ánh vòng đời của các dịch vụ.

## 6. Tổng kết
Dependency Injection là một khía cạnh quan trọng trong ASP.NET Core, giúp bạn quản lý các phụ thuộc một cách dễ dàng và hiệu quả. Hiểu rõ cách hoạt động của các vòng đời dịch vụ như Transient, Scoped, và Singleton sẽ giúp bạn áp dụng DI một cách chính xác trong ứng dụng của mình.

