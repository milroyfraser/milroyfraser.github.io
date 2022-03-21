# .Net Core Web API Acceptance Testing

> I am NOT an experienced .Net developer. In this quarantined vacation, my fun side project was to implement a simple RESTful APIs in 3 different languages which I am not familiar with. So the first choice was .Net Core.

<br>

While I was implementing the API, I enjoyed tolts of cool stuff comes out of the box with .Net Core Web API such as auto-generated API docs with [Swagger UI](https://swagger.io/tools/swagger-ui/), content negotiation support, response helpers and many more.

<br>

The only thing that I was not happy with is the amount of work need to be done just to get started writing E2E/Acceptance test (make a request to an endpoint and assert response)

<br>

After a few days of research, I could not find an easy way to seed test data to the DB. So I implemented my own way of seeding the DB influence by [Laravel](https://laravel.com/). 

<br>

The basic idea is accessing DBContext directly from the test and seed DB with fake data with the help of [Faker.Net](https://www.nuget.org/packages/Faker.Net/).

<br>

#### BaseIntegrationTest class

```c#
using System;
using System.Linq;
using System.Net.Http;
using Blink.DbContexts;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.AspNetCore.TestHost;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;

namespace Blink.AcceptanceTests.Controllers
{
    public class BaseIntegrationTest : IDisposable
    {
        protected readonly HttpClient TestClient;
        protected readonly BlinkContext DbContext;

        protected BaseIntegrationTest()
        {
            var appFactory = new WebApplicationFactory<Startup>()
                .WithWebHostBuilder(ConfigureWebHost);
            
            DbContext = appFactory.Services.CreateScope().ServiceProvider.GetService<BlinkContext>();
            TestClient = appFactory.CreateClient();
        }

        private void ConfigureWebHost(IWebHostBuilder builder)
        {
            builder.ConfigureTestServices(services =>
            {
                var descriptor = services.SingleOrDefault(d => d.ServiceType == typeof(DbContextOptions<BlinkContext>));
                if (descriptor != null)
                    services.Remove(descriptor);

                services.AddDbContext<BlinkContext>(options =>
                    options.UseInMemoryDatabase("blink-test-db"));
            });
        }

        public void Dispose()
        {
            DbContext.Database.EnsureDeleted();
        }
    }
}
``` 

<br>

#### AuthorCollectionsControllerTest

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using Blink.Dtos;
using Blink.Entities;
using Blink.TestSeed.Seeders;
using FluentAssertions;
using Newtonsoft.Json;
using Xunit;

namespace Blink.AcceptanceTests.Controllers
{
    public class AuthorCollectionsControllerTest : BaseIntegrationTest
    {
        [Fact]
        public async void can_fetch_a_list_of_authors_by_ids()
        {
            // Arrange
            var authors = CreateAuthor(2);

            // Act
            var authorIds = String.Join(",", authors.Select(a => a.Id).ToList());
            var response = await TestClient.GetAsync($"api/author-collections/({authorIds})");
            var data = await response.Content.ReadAsAsync<List<AuthorDto>>();

            // Assert
            response.StatusCode.Should().Be((int) HttpStatusCode.OK);
            data.Should().HaveCount(2);
            data.First().Id.Should().Be(authors.First().Id);
            data.First().Name.Should().Be(authors.First().Name);
        }
        
        [Fact]
        public async void can_create_a_list_of_authors()
        {
            // Arrange
            var authorsToBeCreated = new List<AuthorDto>
            {
                new AuthorDto() {Name = "Bill"},
                new AuthorDto() {Name = "Steve"},
            };

            var serializedAuthors = JsonConvert.SerializeObject(authorsToBeCreated, Formatting.Indented);
            HttpContent content = new StringContent(serializedAuthors);

            // Act
            content.Headers.ContentType = MediaTypeHeaderValue.Parse("application/json");
            var response = await TestClient.PostAsync("api/author-collections", content); 
            var data = await response.Content.ReadAsAsync<List<AuthorDto>>();

            // Assert
            response.StatusCode.Should().Be((int) HttpStatusCode.Created);
            
            data.Should().HaveCount(2);
            data.First().Name.Should().Be(authorsToBeCreated.First().Name);
            
            var ids = String.Join(",", data.Select(a => a.Id).ToList());
            response.Headers.Location.Should().Be($"http://localhost/api/author-collections/({ids})");
        }

        private List<Author> CreateAuthor(int count)
        {
            return (new AuthorSeeder(DbContext)).Create(count);
        }
    }
}
```

You can find the code [here](https://github.com/apichef/blink-restful-api-dot-net-core). If you see any problems with this approach. Or ways to improve the code, please let me know on GitHub, Facebook, Twitter or any way you can reach me.

<br>

## Credits

Finally, I would like to thank a few people who shared their knowledge with me:

<br>

Via Youtube:
- [Tim Corey](https://www.youtube.com/user/IAmTimCorey)
- [Nick Chapsas](https://www.youtube.com/user/ElfocrashDev)
- [Les Jackson](https://www.youtube.com/user/binarythistle)

<br>

Via Zoom:
- [Fiqri Ismail](https://www.linkedin.com/in/fiqriismail/)
- [Seminda Rajapaksha](https://www.linkedin.com/in/seminda/)
