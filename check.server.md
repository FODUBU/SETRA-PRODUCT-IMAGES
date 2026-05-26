require('dotenv').config()

const express = require('express')
const cors = require('cors')
const mongoose = require('mongoose')
const path = require('path')
const helmet = require('helmet')
const morgan = require('morgan')
const rateLimit = require('express-rate-limit')

const initializeSystemSchedules = require('./utils/scheduler')

const app = express()

/* =========================================
   TRUST PROXY (Railway / Reverse Proxies)
========================================= */
app.set('trust proxy', 1)

/* =========================================
   CORS CONFIGURATION
========================================= */
const allowedOrigins = [
  process.env.CORS_ORIGIN,
  process.env.FODUBU_CORS_ORIGIN,
  process.env.SETRA_CORS_ORIGIN,
  process.env.TRACO_CORS_ORIGIN,
  'http://localhost:3000',
  'http://localhost:3001'
].filter(Boolean);

const corsOptions = {
  origin: (origin, callback) => {
    if (!origin) return callback(null, true)

    if (allowedOrigins.includes(origin)) {
      callback(null, true)
    } else {
      callback(new Error('CORS policy blocked this origin'))
    }
  },

  credentials: true,

  methods: [
    'GET',
    'POST',
    'PUT',
    'PATCH',
    'DELETE',
    'OPTIONS'
  ],

  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-App-Id',
    'X-App-Identifier',
    'X-App-Name'
  ]
}

/* =========================================
   GLOBAL MIDDLEWARE
========================================= */
app.use(cors(corsOptions))

app.use(express.json({
  limit: '10mb'
}))

app.use(express.urlencoded({
  extended: true,
  limit: '10mb'
}))

app.use(morgan('combined'))

/* =========================================
   STATIC FILES
========================================= */
app.use(express.static(
  path.join(__dirname, '../public')
))

/* =========================================
   SECURITY HEADERS
========================================= */
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],

        scriptSrc: [
          "'self'",
          "'unsafe-inline'",
          'https://*.vercel-insights.com',
          'https://*.vercel-scripts.com'
        ],

        connectSrc: [
          "'self'",
          'https://*.vercel-insights.com',
          'https://*.speed-insights.com'
        ],

        imgSrc: [
          "'self'",
          'data:',
          'https:'
        ],

        styleSrc: [
          "'self'",
          "'unsafe-inline'"
        ]
      }
    }
  })
)

/* =========================================
   RATE LIMITER
========================================= */
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,

  message: {
    error: 'Too many requests. Please try again later.'
  }
})

app.use('/api/', limiter)

/* =========================================
   SPEED / PERFORMANCE LOGGER
========================================= */
app.use((req, res, next) => {
  const start = Date.now()

  res.on('finish', () => {
    const duration = Date.now() - start

    console.log(
      `[PERF] ${req.method} ${req.originalUrl} - ${duration}ms`
    )

    if (duration > 5000) {
      console.warn(
        `[SLOW REQUEST] ${req.originalUrl} took ${duration}ms`
      )
    }
  })

  next()
})

/* =========================================
   DATABASE CONNECTION
========================================= */
mongoose.connect(process.env.DATABASE_URL)
  .then(() => {
    console.log('[Database] MongoDB connected')
  })
  .catch((err) => {
    console.error('[Database] Connection failed:', err)
  })

/* =========================================
   ROUTES IMPORTS
========================================= */
const appRoutes = require('./routes/app.routes')
const authRoutes = require('./routes/auth.routes')
const gcvRoutes = require('./routes/gcv.routes')
const productRoutes = require('./routes/product.routes')
const orderRoutes = require('./routes/order.routes')
const journeyRoutes = require('./routes/journey.routes')
const servicesRoutes = require('./routes/services.routes')
const paymentRoutes = require('./routes/payment.routes')
const userRoutes = require('./routes/user.routes')
const configRoutes = require('./routes/config.routes')
const stellarRoutes = require('./routes/stellar.routes')
const tokenRoutes = require('./routes/token.routes')
const bookingRoutes = require('./routes/booking.routes')
/* =========================================
   ROOT ROUTE
========================================= */
app.get('/', (req, res) => {
  res.sendFile(
    path.join(__dirname, '../public/index.html')
  )
})

/* =========================================
   HEALTH CHECK
========================================= */
app.get('/api/health', (req, res) => {
  res.status(200).json({
    status: 'ok',
    runtime: 'bun',
    timestamp: new Date().toISOString(),

    ecosystem: [
      'FODUBU Main',
      'SETRA',
      'TRACO'
    ]
  })
})

/* =========================================
   API ROUTES
========================================= */
app.use('/api/auth', authRoutes)

app.use('/api/gcv', gcvRoutes)

app.use('/api/config', configRoutes)

app.use('/api/products', productRoutes)

app.use('/api/orders', orderRoutes)

app.use('/api/journey', journeyRoutes)

app.use('/api/services', servicesRoutes)

app.use('/api/bookings', bookingsRoutes)

app.use('/api/payments', paymentRoutes)

app.use('/api/user', userRoutes)
app.use('api/stellar', stellarRoutes)
app.use('api/token', tokenRoutes)

/* =========================================
   LEGACY ROUTES
========================================= */
app.post(
  '/v1/login',
  require('./routes/legacy/login')
)

app.post(
  '/v1/chat/default',
  require('./routes/legacy/chat')
)

/* =========================================
   WEBHOOKS
========================================= */
app.post('/webhook', (req, res) => {
  console.log('[Webhook] Triggered')

  res.sendStatus(200)
})

/* =========================================
   GLOBAL ERROR HANDLER
========================================= */
app.use((err, req, res, next) => {
  console.error('[Server Error]', err)

  res.status(500).json({
    error: 'Internal server error',
require('dotenv').config()

const express = require('express')
const cors = require('cors')
const mongoose = require('mongoose')
const path = require('path')
const helmet = require('helmet')
const morgan = require('morgan')
const rateLimit = require('express-rate-limit')

const initializeSystemSchedules = require('./utils/scheduler')

const app = express()

/* =========================================
   TRUST PROXY (Railway / Reverse Proxies)
========================================= */
app.set('trust proxy', 1)

/* =========================================
   CORS CONFIGURATION
========================================= */
const allowedOrigins = [
  process.env.CORS_ORIGIN,
  process.env.FODUBU_CORS_ORIGIN,
  process.env.SETRA_CORS_ORIGIN,
  process.env.TRACO_CORS_ORIGIN,
  'http://localhost:3000',
  'http://localhost:3001'
].filter(Boolean);

const corsOptions = {
  origin: (origin, callback) => {
    if (!origin) return callback(null, true)

    if (allowedOrigins.includes(origin)) {
      callback(null, true)
    } else {
      callback(new Error('CORS policy blocked this origin'))
    }
  },

  credentials: true,

  methods: [
    'GET',
    'POST',
    'PUT',
    'PATCH',
    'DELETE',
    'OPTIONS'
  ],

  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-App-Id',
    'X-App-Identifier',
    'X-App-Name'
  ]
}

/* =========================================
   GLOBAL MIDDLEWARE
========================================= */
app.use(cors(corsOptions))

app.use(express.json({
  limit: '10mb'
}))

app.use(express.urlencoded({
  extended: true,
  limit: '10mb'
}))

app.use(morgan('combined'))

/* =========================================
   STATIC FILES
========================================= */
app.use(express.static(
  path.join(__dirname, '../public')
))

/* =========================================
   SECURITY HEADERS
========================================= */
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],

        scriptSrc: [
          "'self'",
          "'unsafe-inline'",
          'https://*.vercel-insights.com',
          'https://*.vercel-scripts.com'
        ],

        connectSrc: [
          "'self'",
          'https://*.vercel-insights.com',
          'https://*.speed-insights.com'
        ],

        imgSrc: [
          "'self'",
          'data:',
          'https:'
        ],

        styleSrc: [
          "'self'",
          "'unsafe-inline'"
        ]
      }
    }
  })
)

/* =========================================
   RATE LIMITER
========================================= */
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,

  message: {
    error: 'Too many requests. Please try again later.'
  }
})

app.use('/api/', limiter)

/* =========================================
   SPEED / PERFORMANCE LOGGER
========================================= */
app.use((req, res, next) => {
  const start = Date.now()

  res.on('finish', () => {
    const duration = Date.now() - start

    console.log(
      `[PERF] ${req.method} ${req.originalUrl} - ${duration}ms`
    )

    if (duration > 5000) {
      console.warn(
        `[SLOW REQUEST] ${req.originalUrl} took ${duration}ms`
      )
    }
  })

  next()
})

/* =========================================
   DATABASE CONNECTION
========================================= */
mongoose.connect(process.env.DATABASE_URL)
  .then(() => {
    console.log('[Database] MongoDB connected')
  })
  .catch((err) => {
    console.error('[Database] Connection failed:', err)
  })

/* =========================================
   ROUTES IMPORTS
========================================= */
const appRoutes = require('./routes/app.routes')
const authRoutes = require('./routes/auth.routes')
const gcvRoutes = require('./routes/gcv.routes')
const productRoutes = require('./routes/product.routes')
const orderRoutes = require('./routes/order.routes')
const journeyRoutes = require('./routes/journey.routes')
const servicesRoutes = require('./routes/services.routes')
const paymentRoutes = require('./routes/payment.routes')
const userRoutes = require('./routes/user.routes')
const configRoutes = require('./routes/config.routes')
const stellarRoutes = require('./routes/stellar.routes')
const tokenRoutes = require('./routes/token.routes')
const bookingRoutes = require('./routes/booking.routes')
/* =========================================
   ROOT ROUTE
========================================= */
app.get('/', (req, res) => {
  res.sendFile(
    path.join(__dirname, '../public/index.html')
  )
})

/* =========================================
   HEALTH CHECK
========================================= */
app.get('/api/health', (req, res) => {
  res.status(200).json({
    status: 'ok',
    runtime: 'bun',
    timestamp: new Date().toISOString(),

    ecosystem: [
      'FODUBU Main',
      'SETRA',
      'TRACO'
    ]
  })
})

/* =========================================
   API ROUTES
========================================= */
app.use('/api/auth', authRoutes)

app.use('/api/gcv', gcvRoutes)

app.use('/api/config', configRoutes)

app.use('/api/products', productRoutes)

app.use('/api/orders', orderRoutes)

app.use('/api/journey', journeyRoutes)

app.use('/api/services', servicesRoutes)

app.use('/api/bookings', bookingsRoutes)

app.use('/api/payments', paymentRoutes)

app.use('/api/user', userRoutes)
app.use('api/stellar', stellarRoutes)
app.use('api/token', tokenRoutes)

/* =========================================
   LEGACY ROUTES
========================================= */
app.post(
  '/v1/login',
  require('./routes/legacy/login')
)

app.post(
  '/v1/chat/default',
  require('./routes/legacy/chat')
)

/* =========================================
   WEBHOOKS
========================================= */
app.post('/webhook', (req, res) => {
  console.log('[Webhook] Triggered')

  res.sendStatus(200)
})

/* =========================================
   GLOBAL ERROR HANDLER
========================================= */
app.use((err, req, res, next) => {
  console.error('[Server Error]', err)

  res.status(500).json({
    error: 'Internal server error',

    message:
      process.env.NODE_ENV === 'development'
        ? err.message
        : undefined
  })
})
/* =========================================
   START SERVER
========================================= */
const PORT = process.env.PORT || 3001;

app.listen(PORT, '0.0.0.0', async () => {
  console.log(`🚀 FODUBU API running on port ${PORT}`);
  
  console.log(
    `🌍 Environment: if (isProduction) {
   // production logic`
  );

  try {
    await initializeSystemSchedules();
    console.log('✅ System schedules initialized');
  } catch (error) {
    console.error('❌ Failed to initialize schedules:', error.message);
  }
});

    message:
      process.env.NODE_ENV === 'development'
        ? err.message
        : undefined
  })
})
/* =========================================
   START SERVER
========================================= */
const PORT = process.env.PORT || 3001;

app.listen(PORT, '0.0.0.0', async () => {
  console.log(`🚀 FODUBU API running on port ${PORT}`);
  
  console.log(
    `🌍 Environment: if (isProduction) {
   // production logic`
  );

  try {
    await initializeSystemSchedules();
    console.log('✅ System schedules initialized');
  } catch (error) {
    console.error('❌ Failed to initialize schedules:', error.message);
  }
});

