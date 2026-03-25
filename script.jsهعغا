const STORAGE_KEY = 'constructionManagerAppData';

const defaultData = {
  projects: [],
  expenses: [],
  payments: []
};

let data = loadData();

const views = document.querySelectorAll('.view');
const navButtons = document.querySelectorAll('.nav-btn');
const modals = document.querySelectorAll('.modal');

const projectsCount = document.getElementById('projectsCount');
const paymentsTotal = document.getElementById('paymentsTotal');
const expensesTotal = document.getElementById('expensesTotal');
const profitTotal = document.getElementById('profitTotal');
const recentActivity = document.getElementById('recentActivity');
const projectsList = document.getElementById('projectsList');
const expensesList = document.getElementById('expensesList');
const paymentsList = document.getElementById('paymentsList');
const reportProjectSelect = document.getElementById('reportProjectSelect');
const reportCard = document.getElementById('reportCard');
const todayText = document.getElementById('todayText');

const projectForm = document.getElementById('projectForm');
const expenseForm = document.getElementById('expenseForm');
const paymentForm = document.getElementById('paymentForm');

const expenseProject = document.getElementById('expenseProject');
const paymentProject = document.getElementById('paymentProject');

document.getElementById('expenseDate').valueAsDate = new Date();
document.getElementById('paymentDate').valueAsDate = new Date();

todayText.textContent = new Date().toLocaleDateString('ar-IQ');

document.querySelectorAll('[data-open-modal]').forEach(btn => {
  btn.addEventListener('click', () => openModal(btn.dataset.openModal));
});

document.querySelectorAll('[data-close-modal]').forEach(btn => {
  btn.addEventListener('click', closeAllModals);
});

modals.forEach(modal => {
  modal.addEventListener('click', (e) => {
    if (e.target.classList.contains('modal')) closeAllModals();
  });
});

navButtons.forEach(btn => {
  btn.addEventListener('click', () => switchView(btn.dataset.view));
});

projectForm.addEventListener('submit', (e) => {
  e.preventDefault();

  const project = {
    id: crypto.randomUUID(),
    name: document.getElementById('projectName').value.trim(),
    type: document.getElementById('projectType').value,
    client: document.getElementById('clientName').value.trim(),
    budget: Number(document.getElementById('projectBudget').value),
    notes: document.getElementById('projectNotes').value.trim(),
    createdAt: new Date().toISOString()
  };

  data.projects.unshift(project);
  persist();
  projectForm.reset();
  closeAllModals();
  showToast('تم حفظ المشروع بنجاح');
  renderAll();
  switchView('projects');
});

expenseForm.addEventListener('submit', (e) => {
  e.preventDefault();

  if (!data.projects.length) {
    showToast('أضف مشروع أولاً');
    return;
  }

  const expense = {
    id: crypto.randomUUID(),
    projectId: expenseProject.value,
    type: document.getElementById('expenseType').value,
    amount: Number(document.getElementById('expenseAmount').value),
    date: document.getElementById('expenseDate').value,
    note: document.getElementById('expenseNote').value.trim(),
    createdAt: new Date().toISOString()
  };

  data.expenses.unshift(expense);
  persist();
  expenseForm.reset();
  document.getElementById('expenseDate').valueAsDate = new Date();
  closeAllModals();
  showToast('تم حفظ المصروف');
  renderAll();
  switchView('expenses');
});

paymentForm.addEventListener('submit', (e) => {
  e.preventDefault();

  if (!data.projects.length) {
    showToast('أضف مشروع أولاً');
    return;
  }

  const payment = {
    id: crypto.randomUUID(),
    projectId: paymentProject.value,
    amount: Number(document.getElementById('paymentAmount').value),
    date: document.getElementById('paymentDate').value,
    note: document.getElementById('paymentNote').value.trim(),
    createdAt: new Date().toISOString()
  };

  data.payments.unshift(payment);
  persist();
  paymentForm.reset();
  document.getElementById('paymentDate').valueAsDate = new Date();
  closeAllModals();
  showToast('تم حفظ الدفعة');
  renderAll();
  switchView('payments');
});

reportProjectSelect.addEventListener('change', renderReport);

document.getElementById('exportBtn').addEventListener('click', exportData);
document.getElementById('importInput').addEventListener('change', importData);
document.getElementById('clearDataBtn').addEventListener('click', clearData);

function loadData() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    return saved ? JSON.parse(saved) : structuredClone(defaultData);
  } catch (error) {
    console.error('Load error', error);
    return structuredClone(defaultData);
  }
}

function persist() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
}

function openModal(id) {
  if ((id === 'expenseModal' || id === 'paymentModal') && !data.projects.length) {
    showToast('يجب إضافة مشروع أولاً');
    return;
  }
  document.getElementById(id).classList.add('show');
  updateProjectOptions();
}

function closeAllModals() {
  modals.forEach(modal => modal.classList.remove('show'));
}

function switchView(viewId) {
  views.forEach(view => view.classList.toggle('active', view.id === viewId));
  navButtons.forEach(btn => btn.classList.toggle('active', btn.dataset.view === viewId));
}

function formatMoney(value) {
  return new Intl.NumberFormat('ar-IQ').format(value || 0) + ' د.ع';
}

function getProjectName(id) {
  const project = data.projects.find(p => p.id === id);
  return project ? project.name : 'مشروع غير معروف';
}

function projectPaymentsTotal(projectId) {
  return data.payments
    .filter(item => item.projectId === projectId)
    .reduce((sum, item) => sum + item.amount, 0);
}

function projectExpensesTotal(projectId) {
  return data.expenses
    .filter(item => item.projectId === projectId)
    .reduce((sum, item) => sum + item.amount, 0);
}

function deleteProject(id) {
  if (!confirm('هل تريد حذف هذا المشروع وكل ما يتعلق به؟')) return;

  data.projects = data.projects.filter(project => project.id !== id);
  data.expenses = data.expenses.filter(item => item.projectId !== id);
  data.payments = data.payments.filter(item => item.projectId !== id);
  persist();
  renderAll();
  showToast('تم حذف المشروع');
}

function deleteExpense(id) {
  if (!confirm('حذف هذا المصروف؟')) return;
  data.expenses = data.expenses.filter(item => item.id !== id);
  persist();
  renderAll();
  showToast('تم حذف المصروف');
}

function deletePayment(id) {
  if (!confirm('حذف هذه الدفعة؟')) return;
  data.payments = data.payments.filter(item => item.id !== id);
  persist();
  renderAll();
  showToast('تم حذف الدفعة');
}

function renderDashboard() {
  const totalPayments = data.payments.reduce((sum, item) => sum + item.amount, 0);
  const totalExpenses = data.expenses.reduce((sum, item) => sum + item.amount, 0);
  const totalProfit = totalPayments - totalExpenses;

  projectsCount.textContent = data.projects.length;
  paymentsTotal.textContent = formatMoney(totalPayments);
  expensesTotal.textContent = formatMoney(totalExpenses);
  profitTotal.textContent = formatMoney(totalProfit);

  const activities = [
    ...data.payments.map(item => ({
      type: 'دفعة',
      amount: item.amount,
      date: item.date,
      projectId: item.projectId,
      createdAt: item.createdAt
    })),
    ...data.expenses.map(item => ({
      type: 'مصروف',
      amount: item.amount,
      date: item.date,
      projectId: item.projectId,
      createdAt: item.createdAt
    }))
  ].sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt)).slice(0, 5);

  if (!activities.length) {
    recentActivity.className = 'list-wrap empty-state';
    recentActivity.textContent = 'لا توجد عمليات بعد';
    return;
  }

  recentActivity.className = 'list-wrap';
  recentActivity.innerHTML = activities.map(item => `
    <div class="activity-item">
      <div>
        <strong>${item.type}</strong>
        <div class="muted">${getProjectName(item.projectId)}</div>
      </div>
      <div>
        <strong>${formatMoney(item.amount)}</strong>
        <div class="muted">${item.date || ''}</div>
      </div>
    </div>
  `).join('');
}

function renderProjects() {
  if (!data.projects.length) {
    projectsList.innerHTML = '<div class="empty-state glass">لا توجد مشاريع مضافة حاليًا</div>';
    return;
  }

  projectsList.innerHTML = data.projects.map(project => {
    const income = projectPaymentsTotal(project.id);
    const expense = projectExpensesTotal(project.id);
    const net = income - expense;

    return `
      <article class="project-card glass">
        <div class="card-top">
          <div>
            <h3>${project.name}</h3>
            <div class="muted">الزبون: ${project.client}</div>
          </div>
          <span class="tag">${project.type}</span>
        </div>

        <div class="inline-stats">
          <div class="mini-box">
            <span>المبلغ الكلي</span>
            <strong>${formatMoney(project.budget)}</strong>
          </div>
          <div class="mini-box">
            <span>الداخل</span>
            <strong>${formatMoney(income)}</strong>
          </div>
          <div class="mini-box">
            <span>الخارج</span>
            <strong>${formatMoney(expense)}</strong>
          </div>
          <div class="mini-box">
            <span>الصافي</span>
            <strong>${formatMoney(net)}</strong>
          </div>
        </div>

        <div class="card-actions">
          <button class="small-btn" onclick="openProjectReport('${project.id}')">عرض التقرير</button>
          <button class="small-btn danger" onclick="deleteProject('${project.id}')">حذف</button>
        </div>
      </article>
    `;
  }).join('');
}

function renderExpenses() {
  if (!data.expenses.length) {
    expensesList.innerHTML = '<div class="empty-state glass">لا توجد مصاريف مسجلة</div>';
    return;
  }

  expensesList.innerHTML = data.expenses.map(item => `
    <article class="record-card glass">
      <div class="record-meta">
        <div>
          <h3>${item.type}</h3>
          <div class="muted">${getProjectName(item.projectId)}</div>
        </div>
        <span class="tag">${item.date}</span>
      </div>
      <p>${item.note || 'بدون ملاحظة'}</p>
      <div class="card-actions">
        <strong>${formatMoney(item.amount)}</strong>
        <button class="small-btn danger" onclick="deleteExpense('${item.id}')">حذف</button>
      </div>
    </article>
  `).join('');
}

function renderPayments() {
  if (!data.payments.length) {
    paymentsList.innerHTML = '<div class="empty-state glass">لا توجد دفعات مسجلة</div>';
    return;
  }

  paymentsList.innerHTML = data.payments.map(item => `
    <article class="record-card glass">
      <div class="record-meta">
        <div>
          <h3>دفعة مستلمة</h3>
          <div class="muted">${getProjectName(item.projectId)}</div>
        </div>
        <span class="tag">${item.date}</span>
      </div>
      <p>${item.note || 'بدون ملاحظة'}</p>
      <div class="card-actions">
        <strong>${formatMoney(item.amount)}</strong>
        <button class="small-btn danger" onclick="deletePayment('${item.id}')">حذف</button>
      </div>
    </article>
  `).join('');
}

function updateProjectOptions() {
  const options = data.projects.map(project => `<option value="${project.id}">${project.name}</option>`).join('');
  expenseProject.innerHTML = options;
  paymentProject.innerHTML = options;
  reportProjectSelect.innerHTML = data.projects.length
    ? data.projects.map(project => `<option value="${project.id}">${project.name}</option>`).join('')
    : '<option value="">لا توجد مشاريع</option>';
}

function renderReport() {
  updateProjectOptions();

  if (!data.projects.length) {
    reportCard.innerHTML = '<div class="empty-state">أضف مشروعًا حتى يظهر التقرير</div>';
    return;
  }

  const projectId = reportProjectSelect.value || data.projects[0].id;
  reportProjectSelect.value = projectId;

  const project = data.projects.find(item => item.id === projectId);
  const income = projectPaymentsTotal(projectId);
  const expense = projectExpensesTotal(projectId);
  const net = income - expense;
  const remaining = (project.budget || 0) - income;

  reportCard.innerHTML = `
    <div class="report-box">
      <h3>${project.name}</h3>
      <p class="muted">الزبون: ${project.client} | النوع: ${project.type}</p>
    </div>
    <div class="report-box"><strong>المبلغ الكلي:</strong> ${formatMoney(project.budget)}</div>
    <div class="report-box"><strong>إجمالي الدفعات:</strong> ${formatMoney(income)}</div>
    <div class="report-box"><strong>إجمالي المصاريف:</strong> ${formatMoney(expense)}</div>
    <div class="report-box"><strong>صافي الربح:</strong> ${formatMoney(net)}</div>
    <div class="report-box"><strong>المتبقي من الزبون:</strong> ${formatMoney(remaining)}</div>
    <div class="report-box"><strong>الملاحظات:</strong> ${project.notes || 'لا توجد ملاحظات'}</div>
  `;
}

function openProjectReport(projectId) {
  switchView('reports');
  updateProjectOptions();
  reportProjectSelect.value = projectId;
  renderReport();
}

function exportData() {
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = 'construction-backup.json';
  link.click();
  URL.revokeObjectURL(link.href);
  showToast('تم تصدير النسخة الاحتياطية');
}

function importData(event) {
  const file = event.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = () => {
    try {
      const imported = JSON.parse(reader.result);
      data = {
        projects: Array.isArray(imported.projects) ? imported.projects : [],
        expenses: Array.isArray(imported.expenses) ? imported.expenses : [],
        payments: Array.isArray(imported.payments) ? imported.payments : []
      };
      persist();
      renderAll();
      showToast('تم استيراد البيانات بنجاح');
    } catch (error) {
      console.error(error);
      showToast('ملف غير صالح');
    }
  };
  reader.readAsText(file);
  event.target.value = '';
}

function clearData() {
  if (!confirm('هل أنت متأكد من حذف جميع البيانات؟')) return;
  data = structuredClone(defaultData);
  persist();
  renderAll();
  showToast('تم حذف جميع البيانات');
}

function showToast(message) {
  const toast = document.getElementById('toast');
  toast.textContent = message;
  toast.classList.add('show');
  clearTimeout(showToast.timer);
  showToast.timer = setTimeout(() => {
    toast.classList.remove('show');
  }, 2200);
}

function renderAll() {
  updateProjectOptions();
  renderDashboard();
  renderProjects();
  renderExpenses();
  renderPayments();
  renderReport();
}

window.deleteProject = deleteProject;
window.deleteExpense = deleteExpense;
window.deletePayment = deletePayment;
window.openProjectReport = openProjectReport;

console.log('1001');

renderAll();
