# glp-stack-learn
Server Monitoring with Grafana Prometheus and Loki

import { Injectable } from '@nestjs/common';
import { collectDefaultMetrics, Histogram, Registry } from 'prom-client';

@Injectable()
export class MetricService {
  private readonly registry: Registry;
  public readonly httpRequestDurationMicroseconds: Histogram<string>;
  constructor() {
    this.registry = new Registry();
    collectDefaultMetrics({ register: this.registry });

    this.httpRequestDurationMicroseconds = new Histogram({
      name: 'http_request_duration_seconds',
      help: 'Duration of HTTP requests in seconds',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.1, 0.5, 1, 1.5, 2, 5],
      registers: [this.registry],
    });
  }
  async getMetrics() {
    return await this.registry.metrics();
  }
  async getMetricsJSON() {
    return await this.registry.getMetricsAsJSON();
  }
  getMetricType(): string {
    return this.registry.contentType;
  }
}
import { Controller, Get } from '@nestjs/common';
import { MetricService } from './metrics.service';

@Controller({
  path: '/metrics',
})
export class MetricController {
  constructor(private readonly metricService: MetricService) {}
  @Get()
  async getMetics() {
    return await this.metricService.getMetrics();
  }

  @Get('json')
  async getMeticsJson() {
    return await this.metricService.getMetricsJSON();
  }
}

