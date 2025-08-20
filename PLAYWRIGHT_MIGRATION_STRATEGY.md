# Migration Strategy: Selenium to Playwright for Appian Testing

## Executive Summary

This document outlines the strategy for migrating the Appian Selenium API project from Selenium WebDriver to Playwright. The migration will modernize the testing infrastructure while preserving valuable Appian-specific testing patterns.

## Current State Analysis

### Project Strengths
- Already includes Playwright dependency (1.53.0)
- Uses modern Java patterns with JUnit 5
- Good separation between driver management and test logic
- Extensive Appian-specific UI component library
- Multi-framework support (Cucumber, FitNesse, Java)

### Current Architecture
- **Multi-module Gradle project** with Java 11
- **Three main modules**: `appian-selenium-api` (core), `cucumber-for-appian`, `fitnesse-for-appian`
- **Driver Management**: `RemoteWebDriverBuilder` handles Chrome, Firefox, Edge
- **Test Framework Integration**: `BaseFixture`, `SitesFixture`, `TempoFixture`
- **Appian-Specific Components**: `TempoButton`, `TempoGrid`, `TempoField`, etc.

## Key API Differences

| **Aspect** | **Selenium** | **Playwright** |
|------------|-------------|----------------|
| **Browser Setup** | `WebDriver driver = new ChromeDriver()` | `Browser browser = playwright.chromium().launch()` |
| **Element Location** | `driver.findElement(By.xpath("//button"))` | `page.getByRole(AriaRole.BUTTON)` |
| **Waits** | `WebDriverWait wait = new WebDriverWait(driver, 10)` | Built-in auto-waiting |
| **Actions** | `element.sendKeys("text")` | `locator.fill("text")` |
| **Assertions** | Custom or third-party | `assertThat(locator).isVisible()` |
| **Context Management** | Single browser instance | Multiple browser contexts |

## Migration Strategy

### Phase 1: Foundation (Weeks 1-2)

#### 1.1 Create Playwright Base Classes
- Replace `RemoteWebDriverBuilder` with `PlaywrightBrowserBuilder`
- Create `PlaywrightBaseFixture` alongside existing `BaseFixture`
- Implement browser context management

**Example Implementation:**
```java
public class PlaywrightBrowserBuilder {
    public static Browser createBrowser(Properties props, String browserType) {
        Playwright playwright = Playwright.create();
        BrowserType.LaunchOptions options = new BrowserType.LaunchOptions()
            .setHeadless(Boolean.parseBoolean(props.getProperty("headless", "false")))
            .setDownloadsPath(Paths.get(props.getProperty("download.directory")));
        
        return switch(browserType) {
            case "chrome" -> playwright.chromium().launch(options);
            case "firefox" -> playwright.firefox().launch(options);
            case "edge" -> playwright.chromium().launch(options); // Edge uses Chromium
            default -> throw new IllegalArgumentException("Unsupported browser: " + browserType);
        };
    }
}
```

#### 1.2 Update Dependencies
- Update `build.gradle` to use latest Playwright version
- Maintain Selenium dependencies during parallel testing phase

### Phase 2: Core Components (Weeks 3-6)

#### 2.1 Migrate UI Component Library
Transform existing Selenium components to Playwright equivalents:

**Current Selenium TempoButton:**
```java
public class TempoButton {
    private WebElement element;
    
    public void click() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(ExpectedConditions.elementToBeClickable(element));
        element.click();
    }
}
```

**Playwright TempoButton:**
```java
public class TempoButton {
    private Locator locator;
    private Page page;
    
    public TempoButton(Page page, String selector) {
        this.page = page;
        this.locator = page.locator(selector);
    }
    
    public void click() {
        // Auto-waiting built-in
        locator.click();
    }
    
    public void waitForVisible() {
        locator.waitFor(new Locator.WaitForOptions().setState(WaitForSelectorState.VISIBLE));
    }
}
```

#### 2.2 Preserve Critical Appian Patterns
- **Working Indicator Waits**: Convert explicit waits to Playwright's `waitFor()` methods
- **XPath Selectors**: Maintain where they work well with Appian's structure
- **Locale Handling**: Ensure date/time localization works with browser contexts

### Phase 3: Test Framework Integration (Weeks 7-10)

#### 3.1 Cucumber Integration
- Update step definitions to use Playwright fixtures
- Maintain scenario structure and feature files
- Convert WebDriver interactions to Page/Locator patterns

#### 3.2 FitNesse Integration
- Update fixture methods to use Playwright
- Preserve wiki-based test authoring capability
- Convert element interactions

#### 3.3 Direct Java Tests
- Update JUnit 5 test classes
- Convert WebDriver setup/teardown to Playwright lifecycle

### Phase 4: Validation & Cleanup (Weeks 11-12)

#### 4.1 Parallel Testing
- Run both Selenium and Playwright tests
- Compare results and performance metrics
- Identify and resolve discrepancies

#### 4.2 Performance Validation
- Measure execution time improvements (expected: 80% reduction)
- Validate parallel execution capabilities
- Assess flakiness reduction

#### 4.3 Cleanup
- Remove Selenium dependencies
- Archive old implementation
- Update documentation

## Expected Benefits

### Performance Improvements
- **80% reduction in test execution time** (based on industry experiences)
- **Reduced flakiness** due to auto-waiting mechanisms
- **Better parallel execution** with browser contexts
- **Improved debugging** with built-in tracing

### Modern Features
- **Native API testing** capability
- **Built-in auto-waiting** eliminates explicit waits
- **Modern web support** for SPAs, Shadow DOM
- **Enhanced debugging** with trace files

## Risk Mitigation

### Technical Risks
1. **XPath Compatibility**: Test all existing XPath selectors
2. **Appian-Specific Waits**: Ensure "Working..." indicators work correctly
3. **Browser Context Isolation**: Validate test isolation

### Mitigation Strategies
1. **Parallel Testing**: Run both frameworks during transition
2. **Incremental Migration**: Start with least complex test cases
3. **Rollback Plan**: Keep Selenium infrastructure until full validation
4. **Comprehensive Testing**: Full regression testing before go-live

## Implementation Checklist

### Phase 1: Foundation
- [ ] Create `PlaywrightBrowserBuilder` class
- [ ] Implement `PlaywrightBaseFixture`
- [ ] Set up browser context management
- [ ] Update Gradle dependencies
- [ ] Create parallel testing setup

### Phase 2: Core Components
- [ ] Migrate `TempoButton` class
- [ ] Migrate `TempoGrid` class
- [ ] Migrate `TempoField` class
- [ ] Convert XPath selectors where beneficial
- [ ] Implement Appian-specific waits

### Phase 3: Framework Integration
- [ ] Update Cucumber step definitions
- [ ] Migrate FitNesse fixtures
- [ ] Convert direct Java tests
- [ ] Update configuration handling

### Phase 4: Validation
- [ ] Run parallel test suites
- [ ] Performance benchmarking
- [ ] Flakiness analysis
- [ ] Documentation updates
- [ ] Team training completion

## Success Criteria

1. **All existing tests pass** with Playwright implementation
2. **Performance improvement** of at least 50% in execution time
3. **Reduced flakiness** compared to current Selenium tests
4. **Team confidence** in maintaining Playwright-based tests
5. **Documentation completeness** for future maintenance

## Timeline Summary

- **Total Duration**: 12 weeks
- **Phase 1** (Foundation): 2 weeks
- **Phase 2** (Core Components): 4 weeks  
- **Phase 3** (Framework Integration): 4 weeks
- **Phase 4** (Validation & Cleanup): 2 weeks

## Next Steps

1. **Stakeholder Review**: Get approval for migration plan
2. **Team Training**: Schedule Playwright training sessions
3. **Environment Setup**: Prepare development environments
4. **Kickoff**: Begin Phase 1 implementation

---

*This document should be reviewed and updated as the migration progresses. Regular checkpoints should be scheduled to assess progress and adjust timelines as needed.*