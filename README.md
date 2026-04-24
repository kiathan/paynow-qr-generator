# paynow-qr-generator

Credits: xkjyeah

Generates static PayNow QR codes for use in garage sales, hawker stalls, pop-ups, and simple counter payments.

[Try it out](https://kiathan.github.io/paynow-qr-generator/)

## Local development

Install dependencies:

```bash
npm install
```

Run the app locally:

```bash
npm run dev
```

Build for production:

```bash
npm run build
```

## GitHub Pages deployment

This project is configured for static deployment under:

`https://kiathan.github.io/paynow-qr-generator/`

The main app lives at:

`https://kiathan.github.io/paynow-qr-generator/`

The dedicated static image page lives at:

`https://kiathan.github.io/paynow-qr-generator/image.html`

## URL parameters

Both the main app and the image page accept query parameters:

- `mode=phone` or `mode=uen`
- `target=...`
- `reference=...` optional
- `amount=...` optional

Examples:

Main app:

`https://kiathan.github.io/paynow-qr-generator/?mode=phone&target=91234567&reference=STALL-01&amount=12.50`

Dedicated image page:

`https://kiathan.github.io/paynow-qr-generator/image.html?mode=phone&target=91234567&reference=STALL-01&amount=12.50`

Legacy compatibility mode on the main page:

`https://kiathan.github.io/paynow-qr-generator/?mode=phone&target=91234567&reference=STALL-01&amount=12.50&view=image`

## Notes

- `phone` mode accepts local 8-digit Singapore mobile numbers and normalizes them to `+65...`
- `uen` mode accepts uppercase alphanumeric UEN values
- Leaving `amount` blank creates an editable-amount QR
- Providing `amount` creates a fixed-amount QR
- `image.html` is the best option for GitHub Pages when you want a cleaner shareable QR-only URL
