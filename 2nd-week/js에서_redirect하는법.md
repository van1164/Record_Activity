window.location.replace("http://example.com");

SVLETE에서 SSR로 처리할때
import { goto } from '$app/navigation';
// ...Your other imports

goto('/redirectpage');