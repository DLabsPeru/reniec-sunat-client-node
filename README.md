# @destiny-labs/reniec-sunat-client

Cliente TypeScript para consumir el microservicio `reniec-sunat` desde Node.js.
Incluye tipos, validacion de DNI/RUC, timeout, cancelacion y errores de API
estructurados. No requiere dependencias en tiempo de ejecucion.

## Requisitos

- Node.js 20 o superior
- TypeScript 5 o superior si el proyecto consumidor usa TypeScript

## Instalacion

El paquete es publico y se distribuye mediante npmjs. No requiere token ni
archivo `.npmrc`:

```bash
npm install @destiny-labs/reniec-sunat-client
```

## Uso rapido

```ts
import {
  ReniecSunatApiError,
  createReniecSunatClient
} from "@destiny-labs/reniec-sunat-client";

const client = createReniecSunatClient({
  timeoutMs: 15_000
});

const health = await client.health();
const person = await client.getPersonByDni("71101328");
const company = await client.getCompanyByRuc("20604633070");

console.log(health.status);
console.log(person.firstNames, person.lastNames);
console.log(company.businessName);

try {
  await client.getCompanyByRuc("20114052311");
} catch (error) {
  if (error instanceof ReniecSunatApiError) {
    console.error(error.statusCode, error.code, error.requestId);
  }
}
```

La URL de produccion ya esta configurada por defecto:

```text
https://api-reniec-sunat.destiny-peru.com
```

Para desarrollo local u otro ambiente:

```ts
const client = createReniecSunatClient({
  baseUrl: "http://localhost:8080",
  headers: {
    "X-Request-ID": "demo-123"
  }
});
```

Si el servicio incorpora autenticacion:

```ts
const client = createReniecSunatClient({
  bearerToken: process.env.RENIEC_SUNAT_TOKEN
});
```

## Metodos

```ts
client.health(options?)
client.getPersonByDni(dni, options?)
client.getCompanyByRuc(ruc, options?)
client.getTodayExchangeRate(options?)
client.getExchangeRateByDate("2026-07-28", options?)
client.getExchangeRatesByMonth(2026, 7, options?)
```

Cada metodo acepta un `AbortSignal` y headers adicionales:

```ts
const controller = new AbortController();

const person = await client.getPersonByDni("71101328", {
  signal: controller.signal,
  headers: {
    "X-Request-ID": crypto.randomUUID()
  }
});
```

## Errores

- `ReniecSunatValidationError`: DNI, RUC, fecha o periodo invalido.
- `ReniecSunatApiError`: la API respondio con HTTP 4xx o 5xx.
- `ReniecSunatTimeoutError`: se alcanzo el timeout configurado.
- `ReniecSunatNetworkError`: no fue posible conectar con la API.

`ReniecSunatApiError` expone `statusCode`, `code`, `requestId`, `path`,
`details` y los helpers `isBadRequest`, `isNotFound`, `isUnauthorized`,
`isBadGateway` e `isServiceUnavailable`.

## Fechas

El cliente mantiene las fechas como cadenas ISO para evitar cambios silenciosos
por zona horaria. Las fechas de negocio usan `YYYY-MM-DD` y los timestamps
mantienen el formato enviado por la API.

## Desarrollo

```bash
npm install
npm run check
npm test
npm run build
```

Para verificar el contenido que se publicara:

```bash
npm pack --dry-run
```

Cada Pull Request hacia `main` ejecuta el workflow CI con Node.js 20 y 24 para
validar tipos, pruebas, build, auditoria de dependencias y contenido del
paquete antes de permitir el merge.

## Publicacion

El proyecto usa Release Please y Conventional Commits para administrar las
versiones automaticamente:

1. Un `feat:` genera una version minor.
2. Un `fix:` genera una version patch.
3. Un `feat!:` o `BREAKING CHANGE:` genera una version major.
4. Release Please crea o actualiza el PR de release.
5. Al fusionar el PR, se crea el tag `vX.Y.Z` y el GitHub Release.
6. El paquete de esa version se prueba, compila y publica publicamente en npmjs.

El historial de versiones se conserva desde `1.0.1`. El paquete se publica
bajo el scope `@destiny-labs` desde la version `1.0.2`. El manifest,
`package.json`, `package-lock.json` y `CHANGELOG.md` se actualizan mediante el
PR de release.

La publicacion utiliza npm Trusted Publishing mediante OIDC. El publicador
confiable debe autorizar el repositorio
`DLabsPeru/reniec-sunat-client-node` y el workflow
`release-please.yml`. Las publicaciones automatizadas no utilizan tokens npm
persistentes. Un registro npm no permite reemplazar una version que ya fue
publicada.

## Migracion desde el scope anterior

El paquete publicado como `@destiny-peru/reniec-sunat-client` no puede cambiar
de scope en npm. Los consumidores deben reemplazarlo por el paquete nuevo:

```bash
npm uninstall @destiny-peru/reniec-sunat-client
npm install @destiny-labs/reniec-sunat-client
```

Los imports deben usar `@destiny-labs/reniec-sunat-client`. El paquete anterior
puede marcarse como obsoleto indicando esta ruta de migracion.
