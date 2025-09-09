# Trip Analytics Stack

## What the stack does

This Docker Compose stack analyzes trip data from a PostgreSQL database and generates summary statistics. The application connects to a PostgreSQL database containing trip records with city, duration, and fare information, then calculates total trips, average fares by city, and top cities by average trip duration. The results are written to both stdout and a JSON file in the output directory.

## Exact commands to run/stop

### Run the stack:
```bash
make up
```
or
```bash
docker compose up --build
```

### Stop the stack:
```bash
make down
```
or
```bash
docker compose down -v
```

### Clean and run everything:
```bash
make all
```
This will clean the output directory, remove containers/volumes, and start fresh.

## Example output

```
=== Summary ===
{
  "total_trips": 6,
  "avg_fare_by_city": [
    {
      "city": "San Francisco",
      "avg_fare": 20.25
    },
    {
      "city": "New York",
      "avg_fare": 19.0
    },
    {
      "city": "Charlotte",
      "avg_fare": 16.25
    }
  ],
  "top_by_minutes": [
    {
      "city": "San Francisco",
      "avg_minutes": 19.5
    },
    {
      "city": "New York",
      "avg_minutes": 17.5
    },
    {
      "city": "Charlotte",
      "avg_minutes": 16.5
    }
  ]
}
```

## Where outputs are written

- **JSON file**: `./out/summary.json` - Contains the complete analysis results in JSON format
- **Console output**: The same data is also printed to stdout for immediate viewing

The `out/` directory is mounted as a volume from the host system, so files persist after containers are stopped.

## Troubleshooting

### Database not ready
If you see "Waiting for database..." messages, the PostgreSQL container is still starting up. The application will retry connecting for up to 20 seconds (10 retries × 2 second delay). If it still fails:
- Check if the database container is running: `docker compose ps`
- Check database logs: `docker compose logs db`
- Ensure no other service is using port 5432

### Permission issues on out/ directory
If you get permission errors when writing to `./out/`:
- Ensure the `out/` directory exists: `mkdir -p out`
- Check write permissions: `ls -la out/`
- On Linux/macOS, you may need to fix ownership: `sudo chown -R $USER:$USER out/`
- Use `make clean` to recreate the directory with proper permissions

### Container build issues
- Clean Docker cache: `docker system prune -a`
- Rebuild from scratch: `make clean && make up`
- Check Docker is running and has sufficient resources

### Connection timeout errors
- Verify environment variables are set correctly in `compose.yml`
- Check if firewall is blocking Docker networking
- Try increasing the connection timeout in `main.py` if needed
