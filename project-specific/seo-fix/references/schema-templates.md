# Schema Templates for Outdoor Butler

## Schema Type Status (Feb 2026)

### Use freely
Organization, LocalBusiness, Service, Article, BlogPosting, BreadcrumbList,
WebSite, WebPage, Person, VideoObject, ImageObject, Event, AggregateRating

### Restricted — government/health only
FAQ (restricted Aug 2023)

### Deprecated — never use
HowTo (removed Sep 2023), SpecialAnnouncement (Jul 2025)

## LocalBusiness (for location + service pages)

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Outdoor Butler",
  "description": "Professional outdoor cleaning and pressure washing services",
  "url": "https://outdoor-butler.com",
  "telephone": "+34644466783",
  "image": "[LOGO_URL]",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "[CITY]",
    "addressRegion": "Alicante",
    "addressCountry": "ES"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": "[LAT]",
    "longitude": "[LONG]"
  },
  "areaServed": {
    "@type": "GeoCircle",
    "geoMidpoint": { "@type": "GeoCoordinates", "latitude": "[LAT]", "longitude": "[LONG]" },
    "geoRadius": "50000"
  },
  "priceRange": "$$",
  "openingHours": "Mo-Sa 08:00-18:00"
}
```

## Service (for service pages)

```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "name": "[SERVICE_NAME]",
  "description": "[SERVICE_DESCRIPTION]",
  "provider": {
    "@type": "LocalBusiness",
    "name": "Outdoor Butler"
  },
  "areaServed": {
    "@type": "State",
    "name": "Alicante"
  },
  "serviceType": "[pressure washing|deck cleaning|roof cleaning|etc.]"
}
```

## Article (for blog posts)

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "[POST_TITLE]",
  "author": {
    "@type": "Organization",
    "name": "Outdoor Butler"
  },
  "datePublished": "[YYYY-MM-DD]",
  "dateModified": "[YYYY-MM-DD]",
  "publisher": {
    "@type": "Organization",
    "name": "Outdoor Butler",
    "logo": { "@type": "ImageObject", "url": "[LOGO_URL]" }
  }
}
```

## BreadcrumbList

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://outdoor-butler.com/" },
    { "@type": "ListItem", "position": 2, "name": "[PARENT]", "item": "[PARENT_URL]" },
    { "@type": "ListItem", "position": 3, "name": "[CURRENT]" }
  ]
}
```
