# Exercise: Mocking APIs

Test a function that calls an external API by mocking `fetch`.

## Goal

Practice mocking with `vi.fn()` and `vi.spyOn()` — the JS equivalents of Python's `unittest.mock`.

## The Code Under Test

Create `weather.ts`:

```typescript
export interface WeatherData {
  city: string;
  temperature: number;
  condition: string;
}

export async function getWeather(city: string): Promise<WeatherData> {
  const response = await fetch(
    `https://api.weather.example.com/v1/current?city=${encodeURIComponent(city)}`
  );

  if (!response.ok) {
    throw new Error(`Weather API error: ${response.status} ${response.statusText}`);
  }

  const data = await response.json();

  return {
    city: data.location.name,
    temperature: data.current.temp_c,
    condition: data.current.condition.text,
  };
}

export async function getWeatherSummary(cities: string[]): Promise<string> {
  const results = await Promise.allSettled(
    cities.map(city => getWeather(city))
  );

  const lines: string[] = [];
  for (const result of results) {
    if (result.status === "fulfilled") {
      const w = result.value;
      lines.push(`${w.city}: ${w.temperature}°C, ${w.condition}`);
    } else {
      lines.push(`Error: ${result.reason.message}`);
    }
  }

  return lines.join("\n");
}
```

## Tasks

Create `weather.test.ts` with these test cases:

### 1. Test successful response

Mock `fetch` to return a successful weather response. Verify the parsed output.

### 2. Test HTTP error (non-200)

Mock `fetch` to return a 404. Verify that `getWeather` throws with the status code.

### 3. Test network error

Mock `fetch` to throw a network error. Verify it propagates.

### 4. Test getWeatherSummary with mixed results

Mock `fetch` to succeed for some cities and fail for others. Verify the summary string.

<details>
<summary>Hint: Mocking fetch</summary>

```typescript
import { vi } from "vitest";

// Mock global fetch
const mockFetch = vi.fn();
global.fetch = mockFetch;

// Set up a successful response
mockFetch.mockResolvedValueOnce({
  ok: true,
  status: 200,
  json: async () => ({
    location: { name: "London" },
    current: { temp_c: 15, condition: { text: "Cloudy" } },
  }),
});
```
</details>

<details>
<summary>Hint: Resetting mocks between tests</summary>

```typescript
import { beforeEach } from "vitest";

beforeEach(() => {
  vi.restoreAllMocks();
});
```
</details>

<details>
<summary>Solution</summary>

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";
import { getWeather, getWeatherSummary } from "./weather.js";

// Mock fetch globally
const mockFetch = vi.fn();
global.fetch = mockFetch;

beforeEach(() => {
  mockFetch.mockReset();
});

function mockWeatherResponse(city: string, temp: number, condition: string) {
  return {
    ok: true,
    status: 200,
    statusText: "OK",
    json: async () => ({
      location: { name: city },
      current: { temp_c: temp, condition: { text: condition } },
    }),
  };
}

describe("getWeather", () => {
  it("returns parsed weather data on success", async () => {
    mockFetch.mockResolvedValueOnce(
      mockWeatherResponse("London", 15, "Partly Cloudy")
    );

    const result = await getWeather("London");

    expect(result).toEqual({
      city: "London",
      temperature: 15,
      condition: "Partly Cloudy",
    });

    // Verify fetch was called with correct URL
    expect(mockFetch).toHaveBeenCalledWith(
      "https://api.weather.example.com/v1/current?city=London"
    );
  });

  it("encodes city names in URL", async () => {
    mockFetch.mockResolvedValueOnce(
      mockWeatherResponse("New York", 20, "Clear")
    );

    await getWeather("New York");

    expect(mockFetch).toHaveBeenCalledWith(
      "https://api.weather.example.com/v1/current?city=New%20York"
    );
  });

  it("throws on HTTP error response", async () => {
    mockFetch.mockResolvedValueOnce({
      ok: false,
      status: 404,
      statusText: "Not Found",
    });

    await expect(getWeather("Atlantis")).rejects.toThrow(
      "Weather API error: 404 Not Found"
    );
  });

  it("throws on 500 server error", async () => {
    mockFetch.mockResolvedValueOnce({
      ok: false,
      status: 500,
      statusText: "Internal Server Error",
    });

    await expect(getWeather("London")).rejects.toThrow("500");
  });

  it("propagates network errors", async () => {
    mockFetch.mockRejectedValueOnce(new Error("Network failure"));

    await expect(getWeather("London")).rejects.toThrow("Network failure");
  });
});

describe("getWeatherSummary", () => {
  it("returns summary for multiple cities", async () => {
    mockFetch
      .mockResolvedValueOnce(mockWeatherResponse("London", 15, "Cloudy"))
      .mockResolvedValueOnce(mockWeatherResponse("Paris", 22, "Sunny"))
      .mockResolvedValueOnce(mockWeatherResponse("Tokyo", 28, "Humid"));

    const summary = await getWeatherSummary(["London", "Paris", "Tokyo"]);

    expect(summary).toContain("London: 15°C, Cloudy");
    expect(summary).toContain("Paris: 22°C, Sunny");
    expect(summary).toContain("Tokyo: 28°C, Humid");
  });

  it("handles mixed success and failure", async () => {
    mockFetch
      .mockResolvedValueOnce(mockWeatherResponse("London", 15, "Cloudy"))
      .mockResolvedValueOnce({
        ok: false,
        status: 404,
        statusText: "Not Found",
      })
      .mockResolvedValueOnce(mockWeatherResponse("Tokyo", 28, "Clear"));

    const summary = await getWeatherSummary(["London", "Atlantis", "Tokyo"]);

    expect(summary).toContain("London: 15°C");
    expect(summary).toContain("Error: Weather API error: 404");
    expect(summary).toContain("Tokyo: 28°C");
  });

  it("handles all failures", async () => {
    mockFetch.mockRejectedValue(new Error("Network down"));

    const summary = await getWeatherSummary(["London", "Paris"]);

    expect(summary).toContain("Error: Network down");
    expect(summary.split("\n")).toHaveLength(2);
  });
});
```

### Python equivalent for comparison

```python
from unittest.mock import patch, AsyncMock

@patch("weather.fetch", new_callable=AsyncMock)
async def test_get_weather_success(mock_fetch):
    mock_fetch.return_value.ok = True
    mock_fetch.return_value.json.return_value = {
        "location": {"name": "London"},
        "current": {"temp_c": 15, "condition": {"text": "Cloudy"}},
    }

    result = await get_weather("London")
    assert result == {"city": "London", "temperature": 15, "condition": "Cloudy"}
```

Key differences:
- Python uses `@patch` decorator or `with patch(...)` context manager
- JS uses `vi.fn()` for mock functions and assigns them directly
- Both support `mockResolvedValue` / `return_value` for return values
- JS `beforeEach` + `mockReset` serves the same role as Python's automatic mock cleanup
</details>
