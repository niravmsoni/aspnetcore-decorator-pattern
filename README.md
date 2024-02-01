# aspnetcore-decorator-pattern
Demonstrates Decorator pattern implementation in .NET Core

- Structural Design Pattern used for dynamically adding behavior to class without modifying it

	public class Weather
	{
		public class Temperature{get;set;}
	}
	
	public interface IWeatherService
	{
		Weather GetWeather(string city);
	}
	
	
	public class WeatherService : IWeatherService
	{
		private readonly HttpClient _httpClient;
		public WeatherService(IHttpClientFactory httpClientFactory)
		{
			_httpClient = httpClientFactory.CreateClient("weather");
		}
		public Weather GetWeather(string city)
		{
			//HttpClient call
			var response = _httpClient.Get("URL");
			return response;
		}
	}
	
	
	public class CachedWeatherService : IWeatherService
	{
		private readonly WeatherService _innerweatherService;
		private readonly IMemoryCache _memoryCache;
		
		public CachedWeatherService(WeatherService innerWeatherService, IMemoryCache cache)
		{
			_innerweatherService = innerWeatherService;
		}
		
		public Weather GetWeather(string city)
		{
			var key = $"weather-{city}";
			
			if (_memoryCache.TryGetValue(key, out var cachedWeather)
			{
				return cachedWeather;
			}
			else
			{
				var weatherFromService = _innerweatherService.GetWeather(city);
				_memoryCache.Set<Weather>(key, weatherFromService, Timespan.FromMinutes(5));
				return weatherFromService;
			}	
		}
	}
	
	
	//Program
	services.AddScoped<WeatherService>();
	//Doing so - Every-time the first call will land up in CachedWeatherService and then it will go to inner weather service
	services.AddScoped<IWeatherService, CachedWeatherService>();
