# Pros of the `wave-shop` telegram bot

## Advanced User Management System

```

# Location: ./core/bot/models.py

from django.db import models
import uuid

class BotUser(models.Model):
    """
    Advanced user management system with referral tracking
    and subscription state management.
    """
    user_id = models.IntegerField(unique=True)
    test_status = models.CharField(max_length=250, blank=True, null=True)
    is_banned = models.BooleanField(default=False)
    status = models.CharField(max_length=255, blank=True, null=True)
    state = models.CharField(max_length=255, blank=True, null=True)
    selected_product_id = models.IntegerField(null=True, blank=True)
    invited_by = models.IntegerField(null=True, blank=True)
    has_received_prize = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def get_referral_link(self, bot_username):
        """Generate unique referral link for user"""
        return f"https://t.me/{bot_username}?start=ref_{self.user_id}"

    def get_referral_stats(self):
        """Get user's referral statistics"""
        return {
            'total_invites': BotUser.objects.filter(invited_by=self.user_id).count(),
            'active_referrals': BotUser.objects.filter(
                invited_by=self.user_id,
                status='active'
            ).count(),
            'pending_rewards': not self.has_received_prize and self.is_eligible_for_reward()
        }

    def is_eligible_for_reward(self):
        """Check if user is eligible for referral reward"""
        if self.has_received_prize:
            return False
        referral = BotUser.objects.filter(user_id=self.invited_by).first()
        return referral and referral.status == 'active'

```

## Advanced Subscription Management

```

# Location: ./core/website/models.py

class SubscriptionManager:
    """
    Manages different subscription types with discount 
    and promotion handling capabilities.
    """
    def __init__(self):
        self.active_promotions = {}
        self.discount_rules = {}

    class Product(models.Model):
        name = models.CharField(max_length=255)
        price = models.DecimalField(decimal_places=0, max_digits=10)
        data_limit = models.IntegerField()
        expire = models.IntegerField(blank=True, null=True)

    class DiscountCode(models.Model):
        code = models.CharField(max_length=255, unique=True, default=uuid.uuid4)
        active = models.BooleanField(default=True)
        discount_percent = models.FloatField()
        valid_from = models.DateTimeField()
        valid_to = models.DateTimeField()
        use_limit = models.IntegerField(default=1)
        times_used = models.IntegerField(default=0)

        def is_valid(self):
            """Validate discount code"""
            if not self.active or self.times_used >= self.use_limit:
                return False
            current_time = timezone.now()
            return self.valid_from <= current_time <= self.valid_to

        def apply_discount(self, original_price):
            """Apply discount to price"""
            if not self.is_valid():
                return original_price
            discount = original_price * (self.discount_percent / 100)
            return original_price - discount

        def use(self):
            """Track usage of discount code"""
            if self.is_valid():
                self.times_used += 1
                self.save()
                return True
            return False

```

## API Management System

```

# Location: ./core/bot/telegram/utils/api_management.py

import requests
import logging
import aiohttp
from typing import Optional, Dict, Any

class APIManager:
    """
    Comprehensive API management system with retry logic
    and error handling.
    """
    def __init__(self, api_url: str):
        self.api_url = api_url
        self.session = None
        self.retry_count = 3
        self.timeout = 30

    async def create_user(self, username: str, data_limit: float, expire: int, access_token: str) -> Optional[Dict[str, Any]]:
        """Create new VPN user with advanced error handling"""
        url = f"{self.api_url}/api/user"
        payload = {
            "username": username,
            "proxies": {"vmess": {}, "vless": {}},
            "inbounds": {"vmess": [], "vless": []},
            "expire": None,
            "data_limit": data_limit * 1024**3,
            "data_limit_reset_strategy": "no_reset",
            "status": "on_hold",
            "on_hold_expire_duration": expire,
        }

        for attempt in range(self.retry_count):
            try:
                async with aiohttp.ClientSession() as session:
                    async with session.post(
                        url,
                        json=payload,
                        headers=self._get_auth_headers(access_token),
                        timeout=self.timeout
                    ) as response:
                        if response.status == 200:
                            return await response.json()
                        elif response.status == 401:
                            access_token = await self.refresh_token()
                            continue
                        else:
                            logging.error(f"API error: {response.status}")
                            return None
            except Exception as e:
                logging.error(f"Request failed (attempt {attempt + 1}/{self.retry_count}): {str(e)}")
                if attempt == self.retry_count - 1:
                    return None
                await asyncio.sleep(2 ** attempt)  # Exponential backoff

    def _get_auth_headers(self, token: str) -> Dict[str, str]:
        """Generate authentication headers"""
        return {
            "accept": "application/json",
            "Content-Type": "application/json",
            "Authorization": f"Bearer {token}"
        }

    async def refresh_token(self) -> Optional[str]:
        """Handle token refresh"""
        try:
            # Token refresh logic here
            pass
        except Exception as e:
            logging.error(f"Token refresh failed: {str(e)}")
            return None

```

## Django Admin Interface

```

# Location: ./core/bot/admin.py

from django.contrib import admin
from django.utils.html import format_html
from django.urls import reverse
from django.utils.safestring import mark_safe

class BotUserAdmin(admin.ModelAdmin):
    """
    Advanced admin interface for user management
    """
    list_display = ('user_id', 'status', 'subscription_status', 'referral_count', 'created_at')
    list_filter = ('status', 'is_banned', 'test_status')
    search_fields = ('user_id', 'status')
    readonly_fields = ('created_at', 'updated_at')
    
    def subscription_status(self, obj):
        """Display subscription status with color coding"""
        if obj.status == 'active':
            color = 'green'
        elif obj.status == 'expired':
            color = 'red'
        else:
            color = 'orange'
        return format_html(
            '<span style="color: {};">{}</span>',
            color,
            obj.status
        )

    def referral_count(self, obj):
        """Show referral count with link to details"""
        count = obj.get_referral_stats()['total_invites']
        url = reverse('admin:bot_botuser_changelist') + f'?invited_by={obj.user_id}'
        return mark_safe(f'<a href="{url}">{count}</a>')

class OrderAdmin(admin.ModelAdmin):
    """
    Advanced order management interface
    """
    list_display = ('order_id', 'user', 'product', 'status', 'created_at')
    list_filter = ('status', 'created_at')
    search_fields = ('order_id', 'user__user_id')
    readonly_fields = ('created_at', 'updated_at')

    def get_queryset(self, request):
        """Optimize query performance"""
        return super().get_queryset(request).select_related(
            'user', 'product'
        ).prefetch_related(
            'user__subscription_set'
        )

class PaymentAdmin(admin.ModelAdmin):
    """
    Payment tracking and management interface
    """
    list_display = ('id', 'amount', 'status', 'timestamp')
    list_filter = ('status', 'timestamp')
    search_fields = ('id', 'order__order_id')
    readonly_fields = ('timestamp',)

    fieldsets = (
        ('Payment Information', {
            'fields': ('amount', 'status', 'timestamp')
        }),
        ('Order Details', {
            'fields': ('order', 'payment_method')
        }),
        ('Additional Information', {
            'fields': ('notes',),
            'classes': ('collapse',)
        }),
    )

```

## User Activity Monitoring

```

# Location: ./core/bot/admin.py

from django.contrib import admin
from django.utils.html import format_html
from django.urls import reverse
from django.utils.safestring import mark_safe

class BotUserAdmin(admin.ModelAdmin):
    """
    Advanced admin interface for user management
    """
    list_display = ('user_id', 'status', 'subscription_status', 'referral_count', 'created_at')
    list_filter = ('status', 'is_banned', 'test_status')
    search_fields = ('user_id', 'status')
    readonly_fields = ('created_at', 'updated_at')
    
    def subscription_status(self, obj):
        """Display subscription status with color coding"""
        if obj.status == 'active':
            color = 'green'
        elif obj.status == 'expired':
            color = 'red'
        else:
            color = 'orange'
        return format_html(
            '<span style="color: {};">{}</span>',
            color,
            obj.status
        )

    def referral_count(self, obj):
        """Show referral count with link to details"""
        count = obj.get_referral_stats()['total_invites']
        url = reverse('admin:bot_botuser_changelist') + f'?invited_by={obj.user_id}'
        return mark_safe(f'<a href="{url}">{count}</a>')

class OrderAdmin(admin.ModelAdmin):
    """
    Advanced order management interface
    """
    list_display = ('order_id', 'user', 'product', 'status', 'created_at')
    list_filter = ('status', 'created_at')
    search_fields = ('order_id', 'user__user_id')
    readonly_fields = ('created_at', 'updated_at')

    def get_queryset(self, request):
        """Optimize query performance"""
        return super().get_queryset(request).select_related(
            'user', 'product'
        ).prefetch_related(
            'user__subscription_set'
        )

class PaymentAdmin(admin.ModelAdmin):
    """
    Payment tracking and management interface
    """
    list_display = ('id', 'amount', 'status', 'timestamp')
    list_filter = ('status', 'timestamp')
    search_fields = ('id', 'order__order_id')
    readonly_fields = ('timestamp',)

    fieldsets = (
        ('Payment Information', {
            'fields': ('amount', 'status', 'timestamp')
        }),
        ('Order Details', {
            'fields': ('order', 'payment_method')
        }),
        ('Additional Information', {
            'fields': ('notes',),
            'classes': ('collapse',)
        }),
    )

```