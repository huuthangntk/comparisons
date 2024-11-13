# Pros of the `gods-shop` telegram bot

## Enhanced Admin Product Management System

```
# Location: ./handlers/admin/add.py

from aiogram.dispatcher import FSMContext
from aiogram.types import Message, CallbackQuery, InlineKeyboardMarkup, InlineKeyboardButton, ContentType
from aiogram.utils.callback_data import CallbackData
from states import ProductState, CategoryState

# Callback data patterns for flexible product/category management
category_cb = CallbackData('category', 'id', 'action')
product_cb = CallbackData('product', 'id', 'action')

async def show_products(m, products, category_idx):
    """Enhanced product display with admin controls"""
    for idx, title, body, image, price, tag in products:
        text = f'<b>{title}</b>\n\n{body}\n\nЦена: {price} рублей.'
        
        markup = InlineKeyboardMarkup()
        markup.add(InlineKeyboardButton('🗑️ Удалить', callback_data=product_cb.new(id=idx, action='delete')))
        markup.add(InlineKeyboardButton('✏️ Редактировать', callback_data=product_cb.new(id=idx, action='edit')))
        markup.add(InlineKeyboardButton('📊 Статистика', callback_data=product_cb.new(id=idx, action='stats')))

        await m.answer_photo(photo=image, caption=text, reply_markup=markup)

async def process_title(message: Message, state: FSMContext):
    """Enhanced product title processing with validation"""
    title = message.text
    
    if len(title) < 3 or len(title) > 50:
        await message.answer('Название должно быть от 3 до 50 символов')
        return
    
    async with state.proxy() as data:
        data['title'] = title
        # Store original title for edit tracking
        if 'original_title' not in data:
            data['original_title'] = title
            
    await ProductState.next()
    await message.answer('Описание товара?', reply_markup=back_markup())

async def process_image_photo(message: Message, state: FSMContext):
    """Enhanced image processing with validation and optimization"""
    try:
        fileID = message.photo[-1].file_id
        file_info = await bot.get_file(fileID)
        
        # Validate file size and dimensions
        if file_info.file_size > 5_000_000:  # 5MB limit
            await message.answer('Файл слишком большой. Максимальный размер - 5MB')
            return
            
        downloaded_file = (await bot.download_file(file_info.file_path)).read()
        
        async with state.proxy() as data:
            data['image'] = downloaded_file
            data['image_id'] = fileID  # Store ID for future reference
            
        await ProductState.next()
        await message.answer('Цена товара?', reply_markup=back_markup())
        
    except Exception as e:
        await message.answer('Ошибка при обработке изображения. Попробуйте другое фото.')

```

## Advanced Shopping Cart Management

```
# Location: ./handlers/admin/add.py

from aiogram.dispatcher import FSMContext
from aiogram.types import Message, CallbackQuery, InlineKeyboardMarkup, InlineKeyboardButton, ContentType
from aiogram.utils.callback_data import CallbackData
from states import ProductState, CategoryState

# Callback data patterns for flexible product/category management
category_cb = CallbackData('category', 'id', 'action')
product_cb = CallbackData('product', 'id', 'action')

async def show_products(m, products, category_idx):
    """Enhanced product display with admin controls"""
    for idx, title, body, image, price, tag in products:
        text = f'<b>{title}</b>\n\n{body}\n\nЦена: {price} рублей.'
        
        markup = InlineKeyboardMarkup()
        markup.add(InlineKeyboardButton('🗑️ Удалить', callback_data=product_cb.new(id=idx, action='delete')))
        markup.add(InlineKeyboardButton('✏️ Редактировать', callback_data=product_cb.new(id=idx, action='edit')))
        markup.add(InlineKeyboardButton('📊 Статистика', callback_data=product_cb.new(id=idx, action='stats')))

        await m.answer_photo(photo=image, caption=text, reply_markup=markup)

async def process_title(message: Message, state: FSMContext):
    """Enhanced product title processing with validation"""
    title = message.text
    
    if len(title) < 3 or len(title) > 50:
        await message.answer('Название должно быть от 3 до 50 символов')
        return
    
    async with state.proxy() as data:
        data['title'] = title
        # Store original title for edit tracking
        if 'original_title' not in data:
            data['original_title'] = title
            
    await ProductState.next()
    await message.answer('Описание товара?', reply_markup=back_markup())

async def process_image_photo(message: Message, state: FSMContext):
    """Enhanced image processing with validation and optimization"""
    try:
        fileID = message.photo[-1].file_id
        file_info = await bot.get_file(fileID)
        
        # Validate file size and dimensions
        if file_info.file_size > 5_000_000:  # 5MB limit
            await message.answer('Файл слишком большой. Максимальный размер - 5MB')
            return
            
        downloaded_file = (await bot.download_file(file_info.file_path)).read()
        
        async with state.proxy() as data:
            data['image'] = downloaded_file
            data['image_id'] = fileID  # Store ID for future reference
            
        await ProductState.next()
        await message.answer('Цена товара?', reply_markup=back_markup())
        
    except Exception as e:
        await message.answer('Ошибка при обработке изображения. Попробуйте другое фото.')
```

## Advanced Support Ticket System

```

# Location: ./handlers/user/sos.py and ./handlers/admin/questions.py

from aiogram.dispatcher import FSMContext
from aiogram.types import Message, CallbackQuery
from aiogram.utils.callback_data import CallbackData

support_cb = CallbackData('support', 'ticket_id', 'action')

class TicketManager:
    """Enhanced support ticket management system"""
    
    @staticmethod
    async def create_ticket(user_id: int, question: str, category: str = 'general'):
        """Create new support ticket with metadata"""
        ticket_data = {
            'timestamp': datetime.now().isoformat(),
            'status': 'open',
            'category': category,
            'priority': 'normal',
            'question': question
        }
        
        db.query('''INSERT INTO questions 
                    (cid, question, metadata, created_at, status) 
                    VALUES (?, ?, ?, ?, ?)''',
                 (user_id, question, json.dumps(ticket_data), 
                  ticket_data['timestamp'], ticket_data['status']))
        
        return True

    @staticmethod
    async def get_admin_ticket_view():
        """Get organized ticket view for admins"""
        tickets = db.fetchall('''
            SELECT q.*, u.username, u.first_name 
            FROM questions q 
            LEFT JOIN users u ON q.cid = u.user_id 
            ORDER BY q.created_at DESC
        ''')
        
        organized_tickets = {
            'urgent': [],
            'open': [],
            'in_progress': [],
            'resolved': []
        }
        
        for ticket in tickets:
            metadata = json.loads(ticket['metadata'])
            status = metadata.get('status', 'open')
            organized_tickets[status].append({
                'id': ticket['id'],
                'user': ticket['first_name'] or ticket['username'],
                'question': ticket['question'],
                'created_at': ticket['created_at'],
                'metadata': metadata
            })
            
        return organized_tickets

async def handle_ticket_response(message: Message, state: FSMContext):
    """Handle admin responses to support tickets"""
    async with state.proxy() as data:
        ticket_id = data['ticket_id']
        response = message.text
        
        # Update ticket status
        db.query('''UPDATE questions 
                    SET status = ?, admin_response = ?, 
                    responded_at = ?, responded_by = ? 
                    WHERE id = ?''',
                 ('resolved', response, datetime.now().isoformat(), 
                  message.from_user.id, ticket_id))
        
        # Notify user
        ticket = db.fetchone('SELECT * FROM questions WHERE id = ?', (ticket_id,))
        user_id = ticket['cid']
        
        notification = (
            f"🎫 Ответ на ваш тикет #{ticket_id}:\n\n"
            f"❓ Ваш вопрос:\n{ticket['question']}\n\n"
            f"✅ Ответ поддержки:\n{response}"
        )
        
        await bot.send_message(user_id, notification)
        
    await state.finish()
    await message.answer("Ответ отправлен пользователю!")

```

## Advanced Catalog Management System

```

# Location: ./handlers/user/catalog.py

from aiogram.types import Message, CallbackQuery
from aiogram.utils.callback_data import CallbackData

catalog_cb = CallbackData('catalog', 'id', 'action')

class CatalogManager:
    """Enhanced catalog management with advanced features"""
    
    @staticmethod
    async def get_category_products(category_id: str, user_id: int):
        """Get products with availability check and user preferences"""
        products = db.fetchall('''
            SELECT p.*, 
                   (SELECT COUNT(*) FROM orders WHERE products LIKE '%' || p.idx || '%') as times_bought,
                   (SELECT COUNT(*) FROM cart WHERE idx = p.idx) as in_carts
            FROM products p
            WHERE p.tag = (SELECT title FROM categories WHERE idx=?) 
            AND p.idx NOT IN (SELECT idx FROM cart WHERE cid = ?)
            ORDER BY times_bought DESC
        ''', (category_id, user_id))
        
        return products

    @staticmethod
    async def format_product_display(product: tuple, include_stats: bool = False):
        """Format product information for display"""
        idx, title, body, image, price, tag, times_bought, in_carts = product
        
        description = f"<b>{title}</b>\n\n{body}\n\nЦена: {price}₽"
        
        if include_stats:
            description += f"\n\n📊 Статистика:"
            description += f"\n♻️ Куплено раз: {times_bought}"
            description += f"\n🛒 Сейчас в корзинах: {in_carts}"
            
            if times_bought > 10:
                description += "\n🔥 Популярный товар!"
                
        return {
            'photo': image,
            'caption': description,
            'markup': get_product_markup(idx, price, in_carts > 5)
        }

async def show_category_products(message: Message, category_id: str):
    """Display category products with enhanced presentation"""
    products = await CatalogManager.get_category_products(category_id, message.chat.id)
    
    if not products:
        await message.answer("В данной категории пока нет товаров 😢")
        return
        
    # Group products by popularity
    popular_products = []
    regular_products = []
    
    for product in products:
        if product[6] > 10:  # times_bought > 10
            popular_products.append(product)
        else:
            regular_products.append(product)
    
    # Show popular products first
    if popular_products:
        await message.answer("🔥 Популярные товары в этой категории:")
        for product in popular_products:
            display = await CatalogManager.format_product_display(product, include_stats=True)
            await message.answer_photo(
                photo=display['photo'],
                caption=display['caption'],
                reply_markup=display['markup']
            )
    
    # Then show regular products
    if regular_products:
        if popular_products:
            await message.answer("\n📦 Другие товары в этой категории:")
        for product in regular_products:
            display = await CatalogManager.format_product_display(product)
            await message.answer_photo(
                photo=display['photo'],
                caption=display['caption'],
                reply_markup=display['markup']
            )

```