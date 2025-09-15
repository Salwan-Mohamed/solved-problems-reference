# Example: Docker Container Won't Start After Image Update

**Date:** 2024-09-15  
**Reporter:** John Smith  
**Environment:** Production  
**Severity:** High  
**Tags:** `docker` `container` `deployment` `quick-fix`

## 📋 Problem Summary

After updating the Docker image to version 2.1.0, the application container fails to start with exit code 1. The service was working fine with version 2.0.8.

## 🔍 Problem Details

### Symptoms
- Container starts but immediately exits
- Application not accessible via HTTP
- No logs visible in container output
- Health check failing

### Environment Information
- **OS:** Ubuntu 20.04 LTS
- **Technology Stack:** Docker 24.0.5, Node.js 18, Express.js
- **Versions:** App v2.1.0 (previously v2.0.8)
- **Configuration:** Docker Compose, Nginx reverse proxy

### Error Messages/Logs

```
$ docker logs app-container
Error: Cannot find module '/app/dist/index.js'
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:880:15)
    at Function.Module._load (internal/modules/cjs/loader.js:725:27)
    at Function.executeUserEntryPoint [as runMain] (internal/run_main.js:72:12)
    at internal/main/run_main_module.js:17:47
```

### Screenshots

[Container status showing exit code 1 in Docker Desktop]

## 🔧 Investigation Process

### Steps Taken
1. Checked Docker logs for error messages
2. Compared working v2.0.8 with failing v2.1.0 image
3. Inspected Dockerfile changes between versions
4. Verified build process completed successfully

### Findings
- The build process changed from TypeScript compilation to dist/ folder
- Entry point in Dockerfile still pointed to old location
- New version uses different directory structure

## ✅ Solution

### Root Cause
The Dockerfile entry point was hardcoded to `/app/src/index.js` but the new build process outputs to `/app/dist/index.js`. The application structure changed but the Docker configuration wasn't updated.

### Fix Applied

```dockerfile
# OLD Dockerfile entry point
CMD ["node", "src/index.js"]

# NEW Dockerfile entry point  
CMD ["node", "dist/index.js"]
```

### Step-by-Step Resolution
1. Updated Dockerfile entry point from `src/index.js` to `dist/index.js`
2. Rebuilt Docker image with corrected entry point
3. Deployed updated image to production
4. Verified container starts successfully and application responds

### Configuration Changes

```yaml
# docker-compose.yml - no changes needed
version: '3.8'
services:
  app:
    image: myapp:2.1.0-fixed
    # ... rest of config unchanged
```

## 🧪 Testing

### Verification Steps
1. Container starts without immediate exit
2. HTTP health check returns 200 OK
3. Application logs show successful startup
4. All API endpoints respond correctly

### Test Results
- Container starts successfully ✅
- Health check passes ✅  
- Application fully functional ✅
- No performance regression detected ✅

## 🚨 Prevention

### Monitoring
- Added container restart monitoring alert
- Set up log monitoring for module resolution errors
- Created automated health check with early warning

### Process Changes
- Updated deployment checklist to verify entry points match build output
- Added integration test that validates Docker container startup
- Documented build process changes in release notes

## 📚 Related Resources

- [Docker CMD vs ENTRYPOINT documentation](https://docs.docker.com/engine/reference/builder/#cmd)
- [Node.js module resolution docs](https://nodejs.org/api/modules.html#modules_module_resolution_algorithm)
- [Internal deployment guide - Docker best practices](https://company-wiki.com/docker-best-practices)

## 📝 Notes

This issue could have been caught earlier with proper integration testing that includes container startup validation. The TypeScript build process change was documented but the Docker configuration update was missed in the release checklist.

Consider using Docker multi-stage builds to ensure consistency between development and production environments.

---

**Solution Rating:** ⭐⭐⭐⭐⭐ (Quick fix, clear root cause)  
**Time to Resolve:** 45 minutes  
**Follow-up Required:** No - monitoring and process improvements implemented