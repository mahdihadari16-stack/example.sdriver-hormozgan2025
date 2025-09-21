# 🚗 نقشه راه پیاده‌سازی سیستم امتیازدهی راننده

## 📋 فهرست مطالب
- [مقدمه](#مقدمه)
- [Phase 1: پایه‌گذاری](#phase-1-پایه‌گذاری)
- [Phase 2: امتیازدهی اصلی](#phase-2-امتیازدهی-اصلی)
- [Phase 3: تحلیل عملکرد](#phase-3-تحلیل-عملکرد)
- [Phase 4: گزارش‌گیری پیشرفته](#phase-4-گزارش‌گیری-پیشرفته)
- [Phase 5: استقرار و نظارت](#phase-5-استقرار-و-نظارت)
- [زمان‌بندی کلی](#زمان‌بندی-کلی)
- [منابع مورد نیاز](#منابع-مورد-نیاز)
- [ملاحظات امنیتی](#ملاحظات-امنیتی)

## 🎯 مقدمه

این سند نقشه راه کاملی برای پیاده‌سازی سیستم امتیازدهی و تحلیل عملکرد رانندگان ارائه می‌دهد. پروژه در 5 مرحله اصلی طراحی شده که هر مرحله قابلیت‌های خاصی را به سیستم اضافه می‌کند.

**مدت کل پروژه:** 10-12 هفته
**تیم پیشنهادی:** 1 Android Developer, 1 Backend Developer, 1 UI/UX Designer, 1 DevOps Engineer

---

## 📱 Phase 1: پایه‌گذاری
**مدت زمان:** هفته 1-2

### ✅ اهداف
- تنظیم محیط توسعه
- طراحی پایگاه داده
- ایجاد API های اولیه
- تنظیم CI/CD pipeline

### 🔧 کارهای Frontend (Android)

#### 1.1 تنظیم پروژه اندرید
```kotlin
// build.gradle (Project level)
buildscript {
    ext {
        compose_version = '1.5.8'
        kotlin_version = '1.9.22'
    }
}

// build.gradle (App level)
dependencies {
    implementation "androidx.compose.ui:ui:$compose_version"
    implementation "androidx.compose.material3:material3:1.1.2"
    implementation "androidx.navigation:navigation-compose:2.7.6"
    implementation "androidx.hilt:hilt-navigation-compose:1.1.0"
    implementation "com.squareup.retrofit2:retrofit:2.9.0"
    implementation "androidx.room:room-runtime:2.6.1"
    kapt "androidx.room:room-compiler:2.6.1"
}
```

#### 1.2 ساختار پروژه
```
app/src/main/java/com/company/driverapp/
├── ui/
│   ├── dashboard/
│   ├── rating/
│   ├── analytics/
│   └── components/
├── data/
│   ├── repository/
│   ├── remote/
│   └── local/
├── domain/
│   ├── model/
│   └── usecase/
└── di/
```

#### 1.3 Navigation Setup
```kotlin
@Composable
fun DriverAppNavigation() {
    val navController = rememberNavController()
    NavHost(
        navController = navController,
        startDestination = "dashboard"
    ) {
        composable("dashboard") { HormozganDriverDashboard() }
        composable("performance") { DriverPerformanceScreen() }
        composable("analytics") { DetailedAnalyticsDashboard() }
        composable("monthly_report") { MonthlyPerformanceReport() }
    }
}
```

### 🗄️ کارهای Backend

#### 1.4 انتخاب تکنولوژی
**گزینه A: Spring Boot + PostgreSQL**
```java
// pom.xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
    </dependency>
</dependencies>
```

**گزینه B: Node.js + Express + MongoDB**
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^7.0.0",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.0",
    "cors": "^2.8.5"
  }
}
```

#### 1.5 طراحی Database Schema

##### PostgreSQL Schema
```sql
-- جدول رانندگان
CREATE TABLE drivers (
    id SERIAL PRIMARY KEY,
    driver_id VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    email VARCHAR(100),
    city VARCHAR(50) DEFAULT 'بندرعباس',
    license_number VARCHAR(50),
    vehicle_info JSONB,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- جدول سفرها
CREATE TABLE trips (
    id SERIAL PRIMARY KEY,
    trip_id VARCHAR(50) UNIQUE NOT NULL,
    driver_id VARCHAR(50) REFERENCES drivers(driver_id),
    passenger_id VARCHAR(50),
    start_location JSONB,
    end_location JSONB,
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    fare DECIMAL(10,2),
    distance_km DECIMAL(8,2),
    duration_minutes INTEGER,
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- جدول امتیازدهی
CREATE TABLE ratings (
    id SERIAL PRIMARY KEY,
    trip_id VARCHAR(50) REFERENCES trips(trip_id),
    driver_id VARCHAR(50) REFERENCES drivers(driver_id),
    passenger_id VARCHAR(50),
    overall_rating DECIMAL(2,1) CHECK (overall_rating >= 1 AND overall_rating <= 5),
    cleanliness_rating DECIMAL(2,1),
    politeness_rating DECIMAL(2,1),
    safety_rating DECIMAL(2,1),
    punctuality_rating DECIMAL(2,1),
    route_knowledge_rating DECIMAL(2,1),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- جدول آمار عملکرد
CREATE TABLE performance_metrics (
    id SERIAL PRIMARY KEY,
    driver_id VARCHAR(50) REFERENCES drivers(driver_id),
    date DATE,
    total_trips INTEGER DEFAULT 0,
    completed_trips INTEGER DEFAULT 0,
    cancelled_trips INTEGER DEFAULT 0,
    total_earnings DECIMAL(12,2) DEFAULT 0,
    average_rating DECIMAL(3,2),
    total_distance_km DECIMAL(10,2),
    online_hours DECIMAL(5,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- جدول دستاوردها
CREATE TABLE driver_achievements (
    id SERIAL PRIMARY KEY,
    driver_id VARCHAR(50) REFERENCES drivers(driver_id),
    achievement_type VARCHAR(50),
    achievement_name VARCHAR(100),
    description TEXT,
    earned_date TIMESTAMP,
    is_active BOOLEAN DEFAULT true
);

-- ایندکس‌ها برای بهبود عملکرد
CREATE INDEX idx_driver_ratings ON ratings(driver_id, created_at);
CREATE INDEX idx_performance_date ON performance_metrics(driver_id, date);
CREATE INDEX idx_trip_driver ON trips(driver_id, created_at);
```

#### 1.6 API های پایه
```java
// RatingController.java
@RestController
@RequestMapping("/api/v1/ratings")
public class RatingController {
    
    @GetMapping("/driver/{driverId}")
    public ResponseEntity<DriverRatingResponse> getDriverRating(
        @PathVariable String driverId) {
        // Implementation
    }
    
    @PostMapping("/trip/{tripId}")
    public ResponseEntity<String> submitRating(
        @PathVariable String tripId,
        @RequestBody RatingRequest request) {
        // Implementation
    }
}
```

### 🔄 DevOps Setup

#### 1.7 Docker Configuration
```dockerfile
# Dockerfile for Spring Boot
FROM openjdk:17-jre-slim
COPY target/driver-rating-app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - DB_HOST=postgres
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: driver_rating
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### ✅ خروجی مرحله 1
- [ ] پروژه Android راه‌اندازی شده
- [ ] پایگاه داده طراحی و ایجاد شده
- [ ] API های پایه پیاده‌سازی شده
- [ ] محیط Docker آماده
- [ ] CI/CD pipeline فعال

---

## ⭐ Phase 2: امتیازدهی اصلی
**مدت زمان:** هفته 3-4

### 🎯 اهداف
- پیاده‌سازی سیستم امتیازدهی کامل
- ایجاد رابط کاربری امتیازدهی
- اتصال Frontend به Backend
- تست‌های کامل

### 🎨 UI Components

#### 2.1 Star Rating Component
```kotlin
@Composable
fun StarRatingInput(
    rating: Float,
    onRatingChanged: (Float) -> Unit,
    modifier: Modifier = Modifier
) {
    Row(modifier = modifier) {
        for (i in 1..5) {
            Icon(
                imageVector = if (i <= rating) Icons.Filled.Star else Icons.Outlined.Star,
                contentDescription = null,
                tint = Color(0xFFFFC107),
                modifier = Modifier
                    .size(32.dp)
                    .clickable { onRatingChanged(i.toFloat()) }
            )
        }
    }
}
```

#### 2.2 Rating Form Screen
```kotlin
@Composable
fun RatingFormScreen(
    tripId: String,
    onSubmitRating: (RatingData) -> Unit
) {
    var overallRating by remember { mutableStateOf(5f) }
    var cleanlinessRating by remember { mutableStateOf(5f) }
    var politenessRating by remember { mutableStateOf(5f) }
    var comment by remember { mutableStateOf("") }

    LazyColumn(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        item {
            Text(
                text = "امتیاز دهی به راننده",
                fontSize = 24.sp,
                fontWeight = FontWeight.Bold
            )
        }
        
        item {
            RatingSection(
                title = "امتیاز کلی",
                rating = overallRating,
                onRatingChanged = { overallRating = it }
            )
        }
        
        // سایر بخش‌های امتیازدهی...
        
        item {
            Button(
                onClick = {
                    onSubmitRating(
                        RatingData(
                            tripId = tripId,
                            overallRating = overallRating,
                            cleanlinessRating = cleanlinessRating,
                            politenessRating = politenessRating,
                            comment = comment
                        )
                    )
                },
                modifier = Modifier.fillMaxWidth()
            ) {
                Text("ثبت امتیاز")
            }
        }
    }
}
```

### 💾 Backend Rating System

#### 2.3 Rating Service
```java
@Service
@Transactional
public class RatingService {
    
    @Autowired
    private RatingRepository ratingRepository;
    
    @Autowired
    private DriverRepository driverRepository;
    
    public RatingResponse submitRating(String tripId, RatingRequest request) {
        // 1. اعتبارسنجی داده‌ها
        validateRatingRequest(request);
        
        // 2. ذخیره امتیاز
        Rating rating = new Rating();
        rating.setTripId(tripId);
        rating.setDriverId(request.getDriverId());
        rating.setOverallRating(request.getOverallRating());
        // ... سایر فیلدها
        
        ratingRepository.save(rating);
        
        // 3. به‌روزرسانی میانگین امتیاز راننده
        updateDriverAverageRating(request.getDriverId());
        
        return new RatingResponse("امتیاز با موفقیت ثبت شد");
    }
    
    private void updateDriverAverageRating(String driverId) {
        Double averageRating = ratingRepository
            .calculateAverageRating(driverId);
        
        Driver driver = driverRepository.findByDriverId(driverId);
        driver.setAverageRating(averageRating);
        driverRepository.save(driver);
    }
}
```

#### 2.4 Rating Repository
```java
@Repository
public interface RatingRepository extends JpaRepository<Rating, Long> {
    
    @Query("SELECT AVG(r.overallRating) FROM Rating r WHERE r.driverId = :driverId")
    Double calculateAverageRating(@Param("driverId") String driverId);
    
    @Query("SELECT r FROM Rating r WHERE r.driverId = :driverId ORDER BY r.createdAt DESC")
    Page<Rating> findRecentRatingsByDriver(
        @Param("driverId") String driverId, 
        Pageable pageable
    );
    
    @Query("SELECT AVG(r.overallRating) FROM Rating r WHERE r.createdAt >= :startDate")
    Double calculateCityAverageRating(@Param("startDate") LocalDateTime startDate);
}
```

### 🔌 API Integration

#### 2.5 Retrofit API Service
```kotlin
interface RatingApiService {
    @GET("ratings/driver/{driverId}")
    suspend fun getDriverRating(
        @Path("driverId") driverId: String
    ): Response<DriverRatingResponse>
    
    @POST("ratings/trip/{tripId}")
    suspend fun submitRating(
        @Path("tripId") tripId: String,
        @Body rating: RatingRequest
    ): Response<ApiResponse>
    
    @GET("ratings/driver/{driverId}/details")
    suspend fun getDetailedRatings(
        @Path("driverId") driverId: String,
        @Query("page") page: Int,
        @Query("size") size: Int
    ): Response<DetailedRatingsResponse>
}
```

#### 2.6 Repository Pattern
```kotlin
class RatingRepository @Inject constructor(
    private val apiService: RatingApiService,
    private val localDao: RatingDao
) {
    suspend fun getDriverRating(driverId: String): Result<DriverRating> {
        return try {
            val response = apiService.getDriverRating(driverId)
            if (response.isSuccessful) {
                response.body()?.let {
                    // Cache locally
                    localDao.insertRating(it.toEntity())
                    Result.success(it.toDomain())
                } ?: Result.failure(Exception("Empty response"))
            } else {
                // Try to get from cache
                val cached = localDao.getDriverRating(driverId)
                if (cached != null) {
                    Result.success(cached.toDomain())
                } else {
                    Result.failure(Exception("Network error"))
                }
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 🧪 Testing

#### 2.7 Unit Tests
```kotlin
@ExtendWith(MockitoExtension::class)
class RatingViewModelTest {
    
    @Mock
    lateinit var ratingRepository: RatingRepository
    
    @InjectMocks
    lateinit var ratingViewModel: RatingViewModel
    
    @Test
    fun `should load driver rating successfully`() = runTest {
        // Given
        val driverId = "driver123"
        val expectedRating = DriverRating(4.8f, 150)
        whenever(ratingRepository.getDriverRating(driverId))
            .thenReturn(Result.success(expectedRating))
        
        // When
        ratingViewModel.loadDriverRating(driverId)
        
        // Then
        assertEquals(expectedRating, ratingViewModel.driverRating.value)
    }
}
```

#### 2.8 Integration Tests
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase
class RatingControllerIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldSubmitRatingSuccessfully() {
        // Given
        RatingRequest request = new RatingRequest();
        request.setDriverId("driver123");
        request.setOverallRating(4.5);
        
        // When
        ResponseEntity<String> response = restTemplate.postForEntity(
            "/api/v1/ratings/trip/trip123", 
            request, 
            String.class
        );
        
        // Then
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
    }
}
```

### ✅ خروجی مرحله 2
- [ ] سیستم امتیازدهی کامل
- [ ] UI امتیازدهی طراحی و پیاده‌سازی شده
- [ ] API های امتیازدهی تست شده
- [ ] اتصال موبایل به بک‌اند
- [ ] تست‌های واحد و یکپارچگی

---

## 📊 Phase 3: تحلیل عملکرد
**مدت زمان:** هفته 5-6

### 🎯 اهداف
- تحلیل آماری عملکرد رانندگان
- مقایسه با سایر رانندگان
- تولید گزارش‌های بصری
- پیشنهادات بهبود

### 📈 Analytics Service

#### 3.1 Performance Analytics
```java
@Service
public class PerformanceAnalyticsService {
    
    public DriverPerformanceReport generatePerformanceReport(
        String driverId, 
        LocalDate startDate, 
        LocalDate endDate
    ) {
        // جمع‌آوری داده‌های خام
        List<Trip> trips = tripRepository.findByDriverIdAndDateRange(
            driverId, startDate, endDate
        );
        
        List<Rating> ratings = ratingRepository.findByDriverIdAndDateRange(
            driverId, startDate, endDate
        );
        
        // محاسبه شاخص‌های کلیدی
        PerformanceKPIs kpis = calculateKPIs(trips, ratings);
        
        // تحلیل روند
        List<PerformanceTrend> trends = analyzeTrends(driverId, startDate, endDate);
        
        // مقایسه با میانگین شهر
        ComparisonMetrics comparison = compareWithCityAverage(kpis);
        
        return DriverPerformanceReport.builder()
            .driverId(driverId)
            .kpis(kpis)
            .trends(trends)
            .comparison(comparison)
            .generatedAt(LocalDateTime.now())
            .build();
    }
    
    private PerformanceKPIs calculateKPIs(List<Trip> trips, List<Rating> ratings) {
        return PerformanceKPIs.builder()
            .totalTrips(trips.size())
            .completedTrips(countCompletedTrips(trips))
            .averageRating(calculateAverageRating(ratings))
            .totalEarnings(calculateTotalEarnings(trips))
            .averageResponseTime(calculateAverageResponseTime(trips))
            .cancellationRate(calculateCancellationRate(trips))
            .build();
    }
}
```

#### 3.2 Comparison Service
```java
@Service
public class DriverComparisonService {
    
    public DriverRanking calculateDriverRanking(String driverId, String city) {
        // دریافت آمار راننده
        PerformanceMetrics driverMetrics = getDriverMetrics(driverId);
        
        // دریافت آمار سایر رانندگان شهر
        List<PerformanceMetrics> cityDrivers = getAllCityDriverMetrics(city);
        
        // محاسبه رتبه
        int rank = calculateRank(driverMetrics, cityDrivers);
        int totalDrivers = cityDrivers.size();
        
        // محاسبه درصدی
        double percentile = ((double)(totalDrivers - rank + 1) / totalDrivers) * 100;
        
        return DriverRanking.builder()
            .driverId(driverId)
            .rank(rank)
            .totalDrivers(totalDrivers)
            .percentile(percentile)
            .cityAverage(calculateCityAverage(cityDrivers))
            .build();
    }
}
```

### 📊 Data Visualization

#### 3.3 Chart Components
```kotlin
@Composable
fun PerformanceChart(
    data: List<PerformanceDataPoint>,
    modifier: Modifier = Modifier
) {
    val chartEntries = data.mapIndexed { index, point ->
        Entry(index.toFloat(), point.value)
    }
    
    AndroidView(
        modifier = modifier,
        factory = { context ->
            LineChart(context).apply {
                description.isEnabled = false
                setTouchEnabled(true)
                isDragEnabled = true
                setScaleEnabled(true)
                
                val dataSet = LineDataSet(chartEntries, "عملکرد").apply {
                    color = Color.BLUE
                    setCircleColor(Color.BLUE)
                    lineWidth = 2f
                    circleRadius = 4f
                }
                
                data = LineData(dataSet)
                invalidate()
            }
        }
    )
}
```

#### 3.4 Performance Dashboard
```kotlin
@Composable
fun PerformanceDashboard(
    viewModel: PerformanceViewModel
) {
    val performanceData by viewModel.performanceData.collectAsState()
    val comparisonData by viewModel.comparisonData.collectAsState()
    
    LazyColumn {
        item {
            PerformanceOverviewCard(performanceData)
        }
        
        item {
            PerformanceChart(
                data = performanceData.trends,
                modifier = Modifier.height(200.dp)
            )
        }
        
        item {
            ComparisonSection(comparisonData)
        }
        
        item {
            ImprovementSuggestions(performanceData.suggestions)
        }
    }
}
```

### 🤖 Improvement Suggestions

#### 3.5 Suggestion Engine
```java
@Service
public class ImprovementSuggestionService {
    
    public List<ImprovementSuggestion> generateSuggestions(String driverId) {
        DriverPerformanceReport report = performanceService
            .generatePerformanceReport(driverId, LocalDate.now().minusMonths(1), LocalDate.now());
        
        List<ImprovementSuggestion> suggestions = new ArrayList<>();
        
        // تحلیل امتیاز
        if (report.getKpis().getAverageRating() < 4.5) {
            suggestions.add(createRatingImprovementSuggestion(report));
        }
        
        // تحلیل زمان پاسخ
        if (report.getKpis().getAverageResponseTime() > 5.0) {
            suggestions.add(createResponseTimeImprovementSuggestion(report));
        }
        
        // تحلیل نرخ لغو
        if (report.getKpis().getCancellationRate() > 0.05) {
            suggestions.add(createCancellationRateImprovementSuggestion(report));
        }
        
        return suggestions;
    }
    
    private ImprovementSuggestion createRatingImprovementSuggestion(
        DriverPerformanceReport report
    ) {
        return ImprovementSuggestion.builder()
            .type(SuggestionType.RATING_IMPROVEMENT)
            .title("بهبود امتیاز کلی")
            .description("برای افزایش امتیاز، بر نظافت خودرو و رفتار مودبانه تمرکز کنید")
            .actionItems(Arrays.asList(
                "خودرو را روزانه تمیز کنید",
                "با لبخند با مسافران برخورد کنید",
                "از مسیرهای کوتاه‌تر استفاده کنید"
            ))
            .priority(Priority.HIGH)
            .build();
    }
}
```

### ✅ خروجی مرحله 3
- [ ] سیستم تحلیل عملکرد کامل
- [ ] نمودارها و گزارش‌های بصری
- [ ] سیستم مقایسه با سایر رانندگان
- [ ] موتور پیشنهادات بهبود
- [ ] UI تحلیل عملکرد

---

## 📋 Phase 4: گزارش‌گیری پیشرفته
**مدت زمان:** هفته 7-8

### 🎯 اهداف
- گزارش‌گیری ماهانه خودکار
- سیستم دستاوردها و مدال‌ها
- پیش‌بینی عملکرد با ML
- سیستم نوتیفیکیشن

### 📊 Monthly Report Generation

#### 4.1 Report Service
```java
@Service
public class MonthlyReportService {
    
    @Scheduled(cron = "0 0 1 * * ?") // اول هر ماه
    public void generateMonthlyReports() {
        List<String> activeDrivers = driverRepository.findAllActiveDriverIds();
        
        for (String driverId : activeDrivers) {
            try {
                MonthlyReport report = generateMonthlyReport(driverId);
                saveReport(report);
                sendReportNotification(driverId, report);
            } catch (Exception e) {
                logger.error("خطا در تولید گزارش ماهانه برای راننده: " + driverId, e);
            }
        }
    }
    
    public MonthlyReport generateMonthlyReport(String driverId) {
        LocalDate lastMonth = LocalDate.now().minusMonths(1);
        LocalDate startOfMonth = lastMonth.withDayOfMonth(1);
        LocalDate endOfMonth = lastMonth.withDayOfMonth(lastMonth.lengthOfMonth());
        
        // جمع‌آوری داده‌ها
        List<Trip> monthlyTrips = tripRepository.findByDriverIdAndDateRange(
            driverId, startOfMonth, endOfMonth
        );
        
        List<Rating> monthlyRatings = ratingRepository.findByDriverIdAndDateRange(
            driverId, startOfMonth, endOfMonth
        );
        
        // تولید بخش‌های مختلف گزارش
        ExecutiveSummary executiveSummary = generateExecutiveSummary(monthlyTrips, monthlyRatings);
        List<GoalProgress> goalProgress = calculateGoalProgress(driverId, monthlyTrips, monthlyRatings);
        PeakPerformanceAnalysis peakAnalysis = analyzePeakPerformance(monthlyTrips);
        List<ActionPlan> actionPlans = generateActionPlans(driverId, monthlyRatings);
        NextMonthPredictions predictions = generatePredictions(driverId);
        
        return MonthlyReport.builder()
            .driverId(driverId)
            .reportMonth(lastMonth)
            .executiveSummary(executiveSummary)
            .goalProgress(goalProgress)
            .peakAnalysis(peakAnalysis)
            .actionPlans(actionPlans)
            .predictions(predictions)
            .generatedAt(LocalDateTime.now())
            .build();
    }
}
```

#### 4.2 Achievement System
```java
@Service
public class AchievementService {
    
    @EventListener
    public void onRatingSubmitted(RatingSubmittedEvent event) {
        checkForNewAchievements(event.getDriverId());
    }
    
    public void checkForNewAchievements(String driverId) {
        List<AchievementRule> rules = achievementRuleRepository.findAll();
        
        for (AchievementRule rule : rules) {
            if (!hasAchievement(driverId, rule.getAchievementType()) && 
                checkAchievementCriteria(driverId, rule)) {
                
                grantAchievement(driverId, rule);
            }
        }
    }
    
    private boolean checkAchievementCriteria(String driverId, AchievementRule rule) {
        switch (rule.getAchievementType()) {
            case EXCELLENT_DRIVER:
                return hasHighRatingForPeriod(driverId, 4.8, 90); // 4.8+ for 3 months
            
            case POLITENESS_AMBASSADOR:
                return hasHighPolitenessPercentage(driverId, 0.95); // 95% positive feedback
            
            case ROUTE_MASTER:
                return hasHighRouteKnowledgeRating(driverId, 4.7); // 4.7+ route knowledge
            
            case RELIABLE_DRIVER:
                return hasLowCancellationRate(driverId, 0.02, 30); // <2% cancellation for 1 month
            
            case SPEED_DEMON:
                return hasLowAverageResponseTime(driverId, 3.0); // <3 minutes response time
            
            case PASSENGER_FAVORITE:
                return hasFiveStarRatingsCount(driverId, 100); // 100+ five-star ratings
            
            default:
                return false;
        }
    }
    
    private void grantAchievement(String driverId, AchievementRule rule) {
        DriverAchievement achievement = DriverAchievement.builder()
            .driverId(driverId)
            .achievementType(rule.getAchievementType())
            .achievementName(rule.getName())
            .description(rule.getDescription())
            .earnedDate(LocalDateTime.now())
            .isActive(true)
            .build();
        
        achievementRepository.save(achievement);
        
        // ارسال نوتیفیکیشن
        notificationService.sendAchievementNotification(driverId, achievement);
    }
}
```

### 🤖 ML Prediction Service

#### 4.3 Performance Prediction
```java
@Service
public class PredictionService {
    
    private final MLModel performancePredictionModel;
    
    public NextMonthPredictions generatePredictions(String driverId) {
        // جمع‌آوری داده‌های تاریخی
        List<PerformanceMetrics> historicalData = performanceRepository
            .findByDriverIdOrderByDateDesc(driverId, PageRequest.of(0, 12));
        
        if (historicalData.size() < 3) {
            return generateDefaultPredictions();
        }
        
        // آماده‌سازی ویژگی‌ها برای مدل ML
        MLFeatures features = extractFeatures(historicalData);
        
        // پیش‌بینی با مدل
        MLPrediction prediction = performancePredictionModel.predict(features);
        
        return NextMonthPredictions.builder()
            .predictedRating(prediction.getRating())
            .predictedTripsCount(prediction.getTripsCount())
            .predictedEarnings(prediction.getEarnings())
            .achievementProbability(prediction.getAchievementProbability())
            .confidenceScore(prediction.getConfidenceScore())
            .recommendations(generateRecommendations(prediction))
            .build();
    }
    
    private MLFeatures extractFeatures(List<PerformanceMetrics> data) {
        return MLFeatures.builder()
            .averageRatingTrend(calculateTrend(data, PerformanceMetrics::getAverageRating))
            .tripCountTrend(calculateTrend(data, PerformanceMetrics::getTotalTrips))
            .earningsTrend(calculateTrend(data, PerformanceMetrics::getTotalEarnings))
            .seasonality(getCurrentSeason())
            .dayOfWeekPattern(calculateDayOfWeekPattern(data))
            .build();
    }
}
```

### 📱 Notification Service

#### 4.4 Push Notification System
```java
@Service
public class NotificationService {
    
    @Autowired
    private FirebaseMessaging firebaseMessaging;
    
    public void sendAchievementNotification(String driverId, DriverAchievement achievement) {
        try {
            String deviceToken = getDriverDeviceToken(driverId);
            
            Message message = Message.builder()
                .setToken(deviceToken)
                .setNotification(Notification.builder()
                    .setTitle("🏆 دستاورد جدید!")
                    .setBody("مبارک! شما مدال \"" + achievement.getAchievementName() + "\" را کسب کردید")
                    .build())
                .putData("type", "achievement")
                .putData("achievementId", achievement.getId().toString())
                .build();
            
            firebaseMessaging.send(message);
            
        } catch (Exception e) {
            logger.error("خطا در ارسال نوتیفیکیشن دستاورد", e);
        }
    }
    
    public void sendMonthlyReportNotification(String driverId, MonthlyReport report) {
        try {
            String deviceToken = getDriverDeviceToken(driverId);
            
            Message message = Message.builder()
                .setToken(deviceToken)
                .setNotification(Notification.builder()
                    .setTitle("📊 گزارش ماهانه آماده است")
                    .setBody("گزارش عملکرد " + report.getReportMonth() + " شما آماده مشاهده است")
                    .build())
                .putData("type", "monthly_report")
                .putData("reportId", report.getId().toString())
                .build();
            
            firebaseMessaging.send(message);
            
        } catch (Exception e) {
            logger.error("خطا در ارسال نوتیفیکیشن گزارش ماهانه", e);
        }
    }
}
```

#### 4.5 Email Report Service
```java
@Service
public class EmailReportService {
    
    @Autowired
    private JavaMailSender mailSender;
    
    @Autowired
    private ReportPDFGenerator pdfGenerator;
    
    public void sendMonthlyReportEmail(String driverId, MonthlyReport report) {
        try {
            Driver driver = driverRepository.findByDriverId(driverId);
            
            // تولید PDF گزارش
            byte[] pdfReport = pdfGenerator.generatePDF(report);
            
            MimeMessage message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
            
            helper.setTo(driver.getEmail());
            helper.setSubject("گزارش ماهانه عملکرد - " + report.getReportMonth());
            helper.setText(generateEmailBody(driver, report), true);
            
            // اتصال PDF
            helper.addAttachment(
                "monthly_report_" + report.getReportMonth() + ".pdf",
                new ByteArrayResource(pdfReport)
            );
            
            mailSender.send(message);
            
        } catch (Exception e) {
            logger.error("خطا در ارسال ایمیل گزارش ماهانه", e);
        }
    }
    
    private String generateEmailBody(Driver driver, MonthlyReport report) {
        return """
            <div dir="rtl" style="font-family: Tahoma;">
                <h2>سلام %s عزیز</h2>
                <p>گزارش