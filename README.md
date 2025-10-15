/**
 * Sliding Log Rate Limiter (sliding_log_rate_limiter.ts)
 *
 * Implements token-bucket-like sliding window using timestamp logs (per-key).
 * Run: ts-node src/sliding_log_rate_limiter.ts
 */

class SlidingLogLimiter {
  private logs: Map<string, number[]> = new Map();
  private windowMs: number;
  private limit: number;

  constructor(limit: number, windowMs: number) {
    this.limit = limit;
    this.windowMs = windowMs;
  }

  allow(key: string) {
    const now = Date.now();
    const arr = this.logs.get(key) || [];
    // remove old
    const fresh = arr.filter(ts => now - ts < this.windowMs);
    fresh.push(now);
    this.logs.set(key, fresh);
    return fresh.length <= this.limit;
  }

  getCount(key: string) {
    const now = Date.now();
    return (this.logs.get(key) || []).filter(ts => now - ts < this.windowMs).length;
  }
}

// demo
if (require.main === module) {
  const limiter = new SlidingLogLimiter(5, 10000);
  const key = 'user:123';
  for (let i = 0; i < 10; i++) {
    console.log(i, 'allow?', limiter.allow(key), 'count', limiter.getCount(key));
  }
}
