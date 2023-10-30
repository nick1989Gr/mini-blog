# Adding domain name to static IP in GCP using terraform

Let's say you would like to create a LoadBalancer with a static IP so that you can access a cloud storage bucket.
What you actually need is a LoadBalancer(LB), a static IP for the LB, a DNS zone and an A record.
We assume that you have registered your own domain name. In this case we assume you are using cloud domains.

The following terraform code will create all the components that are required.
Pay attention to the DNS zone A record. An A record connects the static IP to the domain name.
Another point that is quite important is to configure in the Cloud domains tab the domain name to use `Cloud DNS` instead of `Google Domains` or `Custom name servers`.

```tf
# Reserve IP address for the load balancer
resource "google_compute_global_address" "static_ip" {
  name = "public-lb-ip"
}


# Create LB backend buckets
resource "google_compute_backend_bucket" "bucket" {
  name        = "bucket backend"
  description = "..."
  bucket_name = var.bucket_name
}


# Create url maps
resource "google_compute_url_map" "url_map" {
  name = "http-lb"

  default_service = google_compute_backend_bucket.bucket.id

  host_rule {
    hosts        = ["*"]
    path_matcher = "path-matcher-2"
  }
  path_matcher {
    name            = "path-matcher-2"
    default_service = google_compute_backend_bucket.bucket.id
  }
}


# Create HTTP target proxies
resource "google_compute_target_http_proxy" "proxy" {
  name    = "http-lb-proxy"
  url_map = google_compute_url_map.url_map.id
}

# Create forwarding rules
resource "google_compute_global_forwarding_rule" "forwarding_rule" {
  name                  = "http-lb-forwarding-rule"
  ip_protocol           = "TCP"
  load_balancing_scheme = "EXTERNAL_MANAGED"
  port_range            = "80"
  target                = google_compute_target_http_proxy.proxy.id
  ip_address            = google_compute_global_address.static_ip.id
  labels                = var.labels
}


# DNS zone and records
resource "google_dns_managed_zone" "dns_zone" {
  name        = "bucket-dns-zone"
  dns_name    = join("", [var.domain_name, "."])
  description = "..."
  labels      = var.labels

  dnssec_config {
    state = "off"
  }
}

resource "google_dns_record_set" "rs_a" {
  name         = google_dns_managed_zone.dns_zone.dns_name
  managed_zone = google_dns_managed_zone.dns_zone.name
  type         = "A"
  ttl          = 300

  rrdatas = [google_compute_global_address.static_ip.address]
}


```