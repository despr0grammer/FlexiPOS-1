<template>
    <div>
        <NuxtLayout name="admin">
            <main class="w-full mx-auto">

                <Head>
                    <Title>Sales Invoice - {{ runtimeConfig.public.appName }}</Title>
                </Head>
                <!-- Sales Details Input -->
                <div>
                    <div class="relative">
                        <div class="absolute inset-0 flex items-center" aria-hidden="true">
                            <div class="w-full border-t border-gray-300" />
                        </div>
                        <div class="relative flex">
                            <h2 class="bg-white text-lg font-semibold">SALES INVOICE</h2>
                        </div>
                    </div>
                    <div class="space-y-4">
                        <!-- Row 1 sales invoice -->
                        <div class="flex items-center space-x-4 mt-8">
                            <div class="w-1/2">
                                <label class="block text-xs font-medium text-gray-700">Invoice Number</label>
                                <input v-model="salesInvoice.invoice_no" type="text" placeholder="invoice number"
                                    class="block w-full mt-3 bg-gray-100 border-gray-300 rounded-md shadow-sm focus:border-gray-500 focus:ring-gray-500 text-sm p-3" />
                            </div>
                            <div class="w-1/2">
                                <label class="block text-xs font-medium text-gray-700">remarks</label>
                                <input v-model="salesInvoice.remarks" type="text" placeholder="remarks"
                                    class="block w-full bg-gray-100 mt-3 border-gray-300 rounded-md shadow-sm focus:border-gray-500 focus:ring-gray-500 text-sm p-3" />
                            </div>
                        </div>
                        <!-- Row 2 sales invoice -->
                        <div class="flex items-center space-x-4 mt-8">
                            <div class="w-1/2">
                                <FormLabel label="Customer" />
                                <FormSelect id="product_category_id" v-model="salesInvoice.customer_id"
                                    :options="state.customers.filter(customer => customer.is_active).map(customer => ({ value: customer.id, label: customer.firstname + ' ' + customer.lastname }))"
                                    placeholder="select customer" required />
                            </div>
                            <div class="w-1/2">
                                <FormLabel label="Sales Representative" />
                                <FormSelect id="employee" v-model="salesInvoice.sales_representative"
                                    :options="state.employees.filter(employee => employee.is_active).map(employee => ({ value: employee.id, label: employee.firstname + ' ' + employee.lastname }))"
                                    placeholder="select sales representative" required />
                            </div>
                        </div>
                        <!-- Row 3 sales invoice -->
                        <div class="flex items-center space-x-4 mt-8">
                            <div class="w-1/2">
                                <FormLabel label="Date" />
                                <input id="date" type="date" v-model="salesInvoice.date"
                                    class="block w-full rounded-md border border-gray-300 shadow-sm focus:border-gray-500 focus:ring-gray-500 text-sm px-3 py-2"
                                    required />
                            </div>
                            <div class="w-1/2">
                                <FormLabel label="Due Date" />
                                <input id="paymentDate" type="date" v-model="salesInvoice.due_date"
                                    class="block w-full rounded-md border border-gray-300 shadow-sm focus:border-gray-500 focus:ring-gray-500 text-sm px-3 py-2"
                                    required />
                            </div>
                        </div>

                        <!-- Row 4 -->
                        <div class="flex items-center space-x-4 mt-10">
                            <div class="w-1/4">
                                <FormLabel label="Prepared By" />
                                <FormNumberField for="prepared_by_id" name="prepared_by_id"
                                    v-model="salesInvoice.prepared_by_id" :placeholder="`${firstname} ${lastname}`"
                                    :value="`${firstname} ${lastname}`" readonly
                                    class="block cursor-default bg-gray-200" />
                            </div>
                            <div class="w-1/4">
                                <FormLabel label="Payment Type" />
                                <FormSelect v-model="salesInvoice.payment_type" :options="[
                                    { value: 'Cash', label: 'Cash' },
                                    { value: 'Cheque', label: 'Cheque' },
                                    { value: 'Bank Transfer', label: 'Bank Transfer' }
                                ]">
                                </FormSelect>
                            </div>
                            <div class="w-1/4">
                                <FormLabel label="Terms (days)" />
                                <FormNumberField class="block" for="terms" name="terms" id="terms"
                                    v-model="salesInvoice.terms" placeholder="terms (days)"
                                    :disabled="salesInvoice.payment_type === 'Cash'" />
                            </div>
                            <div class="w-1/4">
                                <FormLabel label="Is Cancelled" />
                                <FormSelect v-model="salesInvoice.is_cancelled" :options="[
                                    { value: false, label: 'No' },
                                    { value: true, label: 'Yes' },
                                ]">
                                </FormSelect>
                            </div>
                        </div>
                        <div class="relative">
                            <div class="absolute inset-0 flex items-center" aria-hidden="true">
                                <div class="w-full border-t border-gray-300" />
                            </div>
                            <div class="relative flex">
                                <h2 class="bg-white text-lg font-semibold">SALES DETAILS</h2>
                            </div>
                        </div>
                        <!-- Row 5 sales details -->
                        <div class="flex items-center space-x-4 mt-8">
                            <div class="w-1/4">
                                <FormLabel label="Product" />
                                <FormSelect id="product_id" v-model="salesInvoiceDetail.product_id"
                                    :options="state.products.filter(product => product.is_active).map(product => ({ value: product.id, label: product.name }))"
                                    placeholder="select product" required />
                            </div>
                            <div class="w-1/4">
                                <FormLabel label="Quantity" />
                                <FormNumberField for="quantity" name="quantity" v-model="salesInvoiceDetail.quantity"
                                    placeholder="quantity" required />
                            </div>
                            <div class="w-1/4">
                                <FormLabel label="Unit Price" />
                                <FormNumberField for="unit_price" name="unit_price"
                                    v-model="salesInvoiceDetail.unit_price" placeholder="unit price" required />
                            </div>
                            <div class="w-1/4 flex items-end">
                                <button @click="addSales"
                                    class="rounded-md bg-gray-900 px-4 py-2 text-xs font-semibold text-white hover:bg-gray-800">Add
                                    Sales</button>
                            </div>
                        </div>
                    </div>

                    <!-- Sales Table -->
                    <table class="min-w-full divide-y divide-gray-300 mt-8">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Product</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Barcode</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Quantity</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Unit</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Unit Price</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Expiry Date</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Subtotal</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                    Actions</th>
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200">
                            <tr v-for="(sale, index) in sales" :key="index">
                                <td class="px-4 py-2 text-xxs text-gray-700">{{ sale.product_name }}</td>
                                <td class="px-4 py-2 text-xxs text-gray-700">{{ sale.barcode }}</td>
                                <td class="px-4 py-2 text-xxs text-gray-700">{{ sale.quantity }}</td>
                                <td class="px-4 py-2 text-xxs text-gray-700">{{ sale.unit }}</td>
                                <td class="px-4 py-2 text-xxs text-gray-700">${{ sale.unit_price }}</td>
                                <td class="px-4 py-2 text-xxs text-gray-700">{{ sale.expiry_date }}</td>
                                <td class="px-4 py-2 text-xxs text-gray-700">${{ (sale.quantity * sale.unit_price).toFixed(2) }}</td>
                                <td class="px-4 py-2 text-xxs text-gray-700">
                                    <div class="flex space-x-2">
                                        <button @click="editSale(index)" class="text-gray-600 hover:text-gray-900">
                                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20"
                                                fill="currentColor">
                                                <path
                                                    d="M17.414 2.586a2 2 0 00-2.828 0l-10 10V16a1 1 0 001 1h3.414l10-10a2 2 0 000-2.828l-1.586-1.586zM5 13l-1.5 1.5V13h1.5zm4.5-4.5L14 4l2 2-4.5 4.5H9.5V8.5z" />
                                            </svg>
                                        </button>
                                        <button @click="deleteSale(index)" class="text-red-600 hover:text-red-900">
                                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20"
                                                fill="currentColor">
                                                <path fill-rule="evenodd"
                                                    d="M6 2a2 2 0 00-2 2v1H2v2h1v9a2 2 0 002 2h8a2 2 0 002-2V7h1V5h-2V4a2 2 0 00-2-2H6zm4 12a1 1 0 102 0V8a1 1 0 10-2 0v6zm-3-1a1 1 0 002 0V8a1 1 0 10-2 0v5zm8-1a1 1 0 10-2 0V8a1 1 0 102 0v5z"
                                                    clip-rule="evenodd" />
                                            </svg>
                                        </button>
                                    </div>
                                </td>
                            </tr>
                            <tr v-if="sales.length === 0">
                                <td colspan="8" class="px-4 py-2 text-xxs text-gray-500 text-center bg-gray-100">No
                                    sales available.</td>
                            </tr>
                        </tbody>
                    </table>

                    <!-- Save, Print and Cancel Buttons -->
                    <div class="mt-6 flex justify-end space-x-4">
                        <button @click="saveSalesInvoice"
                            class="rounded-md bg-gray-900 px-4 py-2 text-xs font-semibold text-white hover:bg-gray-800">Save</button>
                        <button @click="printInvoice" :disabled="!salesInvoice.invoice_no"
                            class="rounded-md bg-blue-600 px-4 py-2 text-xs font-semibold text-white hover:bg-blue-700 disabled:bg-gray-400 disabled:cursor-not-allowed">
                            Print Invoice
                        </button>
                        <button @click="cancel"
                            class="rounded-md bg-gray-300 px-4 py-2 text-xs font-semibold text-gray-800 hover:bg-gray-400">Cancel</button>
                    </div>
                </div>
            </main>
        </NuxtLayout>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import type { Error } from '@/types/error';
import { customerService } from '~/components/api/admin/CustomerService';
import { useAlert } from '@/composables/alert';
import { useI18n } from 'vue-i18n';
import { employeeService } from '~/components/api/admin/EmployeeService';
import { productService } from '~/components/api/admin/ProductService';
import { salesInvoiceService } from '~/components/api/admin/SalesInvoiceService';
import { salesInvoiceDetailService } from '~/components/api/admin/SalesInvoiceDetailService';

const user_id = computed(() => localStorage.getItem('user_id'));
const firstname = computed(() => localStorage.getItem('firstname'));
const lastname = computed(() => localStorage.getItem('lastname'));

// Alert and i18n setup
const { successAlert } = useAlert();
const { errorAlert } = useAlert();
const { warningAlert } = useAlert();
const { t } = useI18n()

const runtimeConfig = useRuntimeConfig();

interface SalesInvoice {
    invoice_no: string;
    document_no: string;
    prepared_by_id: number;
    customer_id: number;
    sales_representative: number;
    date: string;
    due_date: string;
    terms: number;
    amount: number;
    cancelled_by_id: number;
    approved_by_id: number;
    is_cancelled: boolean;
    is_approved: boolean;
    remarks: string;
    payment_type: string;
}

interface SalesInvoiceDetail {
    sales_invoice_id: number;
    product_id: number;
    product_name: string;
    barcode: string;
    quantity: number;
    unit: string;
    unit_price: number;
    expiry_date: string;
}

interface State {
    customers: any[];
    employees: any[];
    products: any[];
    error: Error | null;
}

const state = reactive<State>({
    customers: [],
    employees: [],
    products: [],
    error: null,
});

const salesInvoice = ref<SalesInvoice>({
    invoice_no: '',
    document_no: '',
    prepared_by_id: 0,
    customer_id: 0,
    sales_representative: 0,
    date: '',
    due_date: '',
    terms: 0,
    amount: 0,
    cancelled_by_id: 0,
    approved_by_id: 0,
    is_cancelled: false,
    is_approved: false,
    remarks: '',
    payment_type: 'Cash',
});

const salesInvoiceDetail = ref<SalesInvoiceDetail>({
    sales_invoice_id: 0,
    product_id: 0,
    product_name: '',
    barcode: '',
    quantity: 0,
    unit: '',
    unit_price: 0,
    expiry_date: '',
});

const sales = ref<SalesInvoiceDetail[]>([]);

function addSales() {
    if (salesInvoiceDetail.value.product_id && salesInvoiceDetail.value.quantity && salesInvoiceDetail.value.unit_price) {
        sales.value.push({ ...salesInvoiceDetail.value });
        // Reset the form
        salesInvoiceDetail.value = {
            sales_invoice_id: 0,
            product_id: 0,
            product_name: '',
            barcode: '',
            quantity: 0,
            unit: '',
            unit_price: 0,
            expiry_date: '',
        };
    } else {
        alert('Please fill in all required fields.');
    }
}

function editSale(index: number) {
    salesInvoiceDetail.value = { ...sales.value[index] };
    sales.value.splice(index, 1);
}

function deleteSale(index: number) {
    sales.value.splice(index, 1);
}

function cancel() {
    navigateTo('/admin/sales_invoice');
}

async function fetchCustomers() {
    try {
        const response = await customerService.getCustomers();
        state.customers = response.data;
    } catch (error: any) {
        state.error = error;
    }
}

async function fetchEmployees() {
    try {
        const response = await employeeService.getEmployees();
        state.employees = response.data;
    } catch (error: any) {
        state.error = error;
    }
}

async function fetchProducts() {
    try {
        const response = await productService.getProducts();
        state.products = response.data;
    } catch (error: any) {
        state.error = error;
    }
}

async function saveSalesInvoice() {
    try {
        if (sales.value.length === 0) {
            console.log('Bill before save:', salesInvoice.value);
            alert('Please add at least one sale item.');
            return;
        }

        // Calculate total amount
        const totalAmount = sales.value.reduce((sum, sale) => sum + (sale.quantity * sale.unit_price), 0);
        salesInvoice.value.amount = totalAmount;
        salesInvoice.value.prepared_by_id = parseInt(user_id.value || '0');

        // Save the sales invoice
        const salesInvoiceResponse = await salesInvoiceService.createSalesInvoice(salesInvoice.value);
        if (salesInvoiceResponse) {
            console.log('Sales invoice saved successfully:', salesInvoiceResponse);
            const salesInvoiceId = salesInvoiceResponse.id;

            // Save each sales invoice detail
            for (const salesInvoiceDetailList of sales.value) {
                salesInvoiceDetailList.sales_invoice_id = salesInvoiceId;
                try {
                    console.log('Saving bill detail:', salesInvoiceDetailList); // Log the detail being saved
                    const result = await salesInvoiceDetailService.createSalesInvoiceDetail(salesInvoiceDetailList);
                    if (result) {
                        console.log('Bill detail saved successfully:', result);
                    } else {
                        console.error('Failed to save bill detail:', salesInvoiceDetailList);
                    }
                } catch (detailError: any) {
                    console.error('Error saving bill detail:', detailError.message);
                }
            }

            alert('Sales invoice has been saved successfully!');
            // navigateTo('/admin/sales_invoice'); // Redirect to the sales invoice list
        } else {
            alert('Failed to save sales invoice.');
        }
        // toggleBillForm(); // Hide the form after save.
    } catch (error: any) {
        console.error('Error saving sales invoice:', error.message);
        alert('An error occurred while saving the sales invoice.');
    }
}

watch(
    () => salesInvoiceDetail.value.product_id,
    (newProductId) => {
        // Find the selected product based on the selected product_id
        const selectedProduct = state.products.find(product => product.id === newProductId);

        // Update the name in billDetail if the selected product exists
        if (selectedProduct) {
            salesInvoiceDetail.value.product_name = selectedProduct.name;
            salesInvoiceDetail.value.barcode = selectedProduct.barcode;
            salesInvoiceDetail.value.unit = selectedProduct.wholesale_unit;
            salesInvoiceDetail.value.expiry_date = selectedProduct.expiry_date; // Update expiry_date if the selected product exists
        } else {
            salesInvoiceDetail.value.product_name = ''; // Reset name if no product is selected
            salesInvoiceDetail.value.barcode = ''; // Reset barcode if no product is selected
            salesInvoiceDetail.value.unit = ''; // Reset unit if no product is selected
            salesInvoiceDetail.value.expiry_date = ''; // Reset expiry_date if no product is selected
        }
    }
);

// Función para imprimir la factura
function printInvoice() {
    try {
        if (!salesInvoice.value.invoice_no) {
            alert('Error: Debe guardar la factura antes de imprimirla.');
            return;
        }

        if (sales.value.length === 0) {
            alert('Error: No hay productos en la factura para imprimir.');
            return;
        }

        // Crear el contenido HTML para imprimir
        const printContent = generateInvoicePrintContent();
        
        // Crear una nueva ventana para imprimir
        const printWindow = window.open('', '_blank', 'width=800,height=600');
        
        if (!printWindow) {
            alert('Error: No se pudo abrir la ventana de impresión. Verifique que no esté bloqueada por el navegador.');
            return;
        }

        printWindow.document.write(printContent);
        printWindow.document.close();
        
        // Esperar a que se cargue el contenido y luego imprimir
        printWindow.onload = () => {
            printWindow.print();
            printWindow.close();
        };
        
    } catch (error) {
        console.error('Error al imprimir:', error);
        alert('Error: No se pudo imprimir la factura. Verifique la configuración de su impresora.');
    }
}

// Función para generar el contenido HTML de la factura
function generateInvoicePrintContent() {
    const customer = state.customers.find(c => c.id === salesInvoice.value.customer_id);
    const salesRep = state.employees.find(e => e.id === salesInvoice.value.sales_representative);
    
    const total = sales.value.reduce((sum, sale) => sum + (sale.quantity * sale.unit_price), 0);
    
    return `
        <!DOCTYPE html>
        <html>
        <head>
            <title>Factura ${salesInvoice.value.invoice_no}</title>
            <style>
                body { font-family: Arial, sans-serif; margin: 20px; }
                .header { text-align: center; margin-bottom: 30px; }
                .invoice-info { margin-bottom: 20px; }
                .invoice-info div { margin: 5px 0; }
                table { width: 100%; border-collapse: collapse; margin: 20px 0; }
                th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
                th { background-color: #f2f2f2; }
                .total { text-align: right; font-weight: bold; margin-top: 20px; }
                .footer { margin-top: 30px; text-align: center; font-size: 12px; }
                @media print {
                    body { margin: 0; }
                    .no-print { display: none; }
                }
            </style>
        </head>
        <body>
            <div class="header">
                <h1>FACTURA DE VENTA</h1>
                <h2>FlexiPOS</h2>
            </div>
            
            <div class="invoice-info">
                <div><strong>Número de Factura:</strong> ${salesInvoice.value.invoice_no}</div>
                <div><strong>Fecha:</strong> ${salesInvoice.value.date}</div>
                <div><strong>Fecha de Vencimiento:</strong> ${salesInvoice.value.due_date}</div>
                <div><strong>Cliente:</strong> ${customer ? customer.firstname + ' ' + customer.lastname : 'N/A'}</div>
                <div><strong>Vendedor:</strong> ${salesRep ? salesRep.firstname + ' ' + salesRep.lastname : 'N/A'}</div>
                <div><strong>Tipo de Pago:</strong> ${salesInvoice.value.payment_type}</div>
                <div><strong>Términos:</strong> ${salesInvoice.value.terms} días</div>
                <div><strong>Observaciones:</strong> ${salesInvoice.value.remarks || 'N/A'}</div>
            </div>
            
            <table>
                <thead>
                    <tr>
                        <th>Producto</th>
                        <th>Código</th>
                        <th>Cantidad</th>
                        <th>Unidad</th>
                        <th>Precio Unitario</th>
                        <th>Subtotal</th>
                    </tr>
                </thead>
                <tbody>
                    ${sales.value.map(sale => `
                        <tr>
                            <td>${sale.product_name}</td>
                            <td>${sale.barcode}</td>
                            <td>${sale.quantity}</td>
                            <td>${sale.unit}</td>
                            <td>$${sale.unit_price.toFixed(2)}</td>
                            <td>$${(sale.quantity * sale.unit_price).toFixed(2)}</td>
                        </tr>
                    `).join('')}
                </tbody>
            </table>
            
            <div class="total">
                <h3>Total: $${total.toFixed(2)}</h3>
            </div>
            
            <div class="footer">
                <p>Gracias por su compra</p>
                <p>Impreso el: ${new Date().toLocaleString()}</p>
            </div>
        </body>
        </html>
    `;
}

onMounted(() => {
    fetchCustomers();
    fetchEmployees();
    fetchProducts();
});
</script>
